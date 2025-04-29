import streamlit as st
import pandas as pd
from collections import defaultdict
import io

st.set_page_config(page_title="ObjectID Conflict Detector", layout="wide")

def process_pcf_file(pcf_file):
    """Process PCF file and return conflicting entries"""
    object_id_dict = defaultdict(set)
    
    # Read the file line by line
    for line in io.TextIOWrapper(pcf_file, encoding='utf-8'):
        line = line.strip()
        if not line.startswith(('#', '[')):
            parts = line.split(',', 2)
            if len(parts) >= 2:
                object_id = parts[0].strip()
                description = parts[1].strip('"').strip()
                object_id_dict[object_id].add(description)
    
    # Find conflicts
    conflicting_entries = []
    for obj_id, descriptions in object_id_dict.items():
        if len(descriptions) > 1:
            for desc in descriptions:
                conflicting_entries.append({'ObjectID': obj_id, 'Description': desc})
    
    return conflicting_entries

def process_config_file(config_file):
    """Process config Excel file and return DPID list"""
    try:
        config_df = pd.read_excel(config_file, sheet_name='Dictionary')
        if 'Unnamed: 1' in config_df.columns:
            return config_df['Unnamed: 1'].dropna().astype(str).str.strip().tolist()
        else:
            st.warning("Column 'Unnamed: 1' not found in config file. Using first column instead.")
            return config_df.iloc[:, 0].dropna().astype(str).str.strip().tolist()
    except Exception as e:
        st.error(f"Error reading config file: {str(e)}")
        return []

def main():
    st.title("📊 ObjectID Conflict Detector")
    st.markdown("""
    This tool analyzes PCF files to detect ObjectIDs with multiple descriptions.
    Upload your files below to check for conflicts.
    """)
    
    # File upload sections
    col1, col2 = st.columns(2)
    
    with col1:
        st.subheader("1. Upload PCF File")
        pcf_file = st.file_uploader("Select PCF file", type=['.pcf'], key='pcf')
        
    with col2:
        st.subheader("2. Upload Configuration File")
        config_file = st.file_uploader("Select Excel config file", 
                                      type=['.xlsx', '.xlsm'], 
                                      key='config')
    
    if st.button("Analyze Files", type="primary"):
        if not pcf_file or not config_file:
            st.warning("Please upload both files to proceed")
            return
            
        with st.spinner("Processing files..."):
            try:
                # Process files
                conflicting_entries = process_pcf_file(pcf_file)
                unique_dpid = process_config_file(config_file)
                
                # Create DataFrames
                df_all = pd.DataFrame(conflicting_entries)
                
                if not df_all.empty:
                    # Filter for Dictionary sheet
                    df_dict = df_all[df_all['ObjectID'].isin(unique_dpid)]
                    
                    # Display results
                    st.success("Analysis complete!")
                    
                    tab1, tab2 = st.tabs(["All Conflicts", "Dictionary Conflicts"])
                    
                    with tab1:
                        st.subheader("All Conflicting ObjectIDs")
                        st.dataframe(df_all, use_container_width=True)
                        st.caption(f"Total conflicts found: {len(df_all)}")
                        
                    with tab2:
                        if not df_dict.empty:
                            st.subheader("Conflicts Matching Configuration")
                            st.dataframe(df_dict, use_container_width=True)
                            st.caption(f"Matching conflicts found: {len(df_dict)}")
                        else:
                            st.warning("No matching ObjectIDs found in configuration")
                            st.info("Check if ObjectID formats match between files")
                    
                    # Create download link
                    output = io.BytesIO()
                    with pd.ExcelWriter(output, engine='xlsxwriter') as writer:
                        df_all.to_excel(writer, sheet_name='ALL', index=False)
                        df_dict.to_excel(writer, sheet_name='Dictionary', index=False)
                    
                    st.download_button(
                        label="📥 Download Results as Excel",
                        data=output.getvalue(),
                        file_name="conflicting_descriptions.xlsx",
                        mime="application/vnd.ms-excel"
                    )
                else:
                    st.success("No conflicting ObjectIDs found - all descriptions are unique!")
                    
            except Exception as e:
                st.error(f"An error occurred: {str(e)}")
                st.exception(e)

if __name__ == "__main__":
    main()
