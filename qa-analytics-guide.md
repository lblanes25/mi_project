# Guide to Adding New Analytics to the Enhanced QA Analytics Framework

Creating new analytics in your Enhanced QA Analytics Framework is a straightforward process once you understand the components involved. This guide will walk you through the steps to add a new analytic from start to finish.

## Step 1: Identify the New Analytic Requirements

Before creating a new analytic, clearly define:

1. **Purpose**: What compliance requirement or quality check is this analytic validating?
2. **Data Sources**: What data will this analytic process?
3. **Validation Rules**: What specific rules determine if records conform?
4. **Reporting Needs**: How should results be grouped and presented?

## Step 2: Update the Data Source Registry

If your analytic uses a new data source, add it to the `data_sources.yaml` file:

```yaml
# In configs/data_sources.yaml
data_sources:
  # Add your new data source
  new_data_source_name:
    type: "report"
    description: "Description of the data source"
    version: "1.0"
    owner: "Responsible Team"
    refresh_frequency: "Monthly"
    file_type: "xlsx"
    file_pattern: "Data_Source_*_{YYYY}{MM}*.xlsx"
    key_columns: ["Primary Key Field"]
    
    validation_rules:
      - type: "row_count_min"
        threshold: 10
        description: "Should have at least 10 records"
      - type: "required_columns"
        columns: ["Required Field 1", "Required Field 2"]
        description: "Critical columns that must be present"
    
    columns_mapping:
      - source: "Original Column Name"
        aliases: ["Alternative Name 1", "Alternative Name 2"]
        target: "Standardized Column Name"
        data_type: "string"  # Options: string, date, integer, float, category

# Update the analytics mapping section
analytics_mapping:
  - data_source: "new_data_source_name"
    analytics: ["XX"]  # Your new analytic ID
```

## Step 3: Add Any Required Reference Data

If your analytic needs reference data (like lookup tables):

1. **Update reference_data.yaml**:

```yaml
# In configs/reference_data.yaml
reference_files:
  New_Reference_Data:
    path: "ref_data/new_reference.xlsx"
    description: "Reference data description"
    format: "dictionary"  # or "dataframe"
    key_column: "Key_Column"
    value_column: "Value_Column"
    version: "2025-Q2"
    max_age_days: 90
    owner: "Data Owner"
    refresh_schedule: "Quarterly"
```

2. **Create the reference file** in the `ref_data` directory

## Step 4: Create the Analytic Configuration

Create a new YAML file for your analytic in the `configs` directory:

```yaml
# configs/qa_XX.yaml (replace XX with your analytic ID)
analytic_id: XX
analytic_name: 'New Analytic Name'
analytic_description: 'Detailed description of what this analytic validates.'

# Reference to data source (from the registry)
data_source:
  name: 'new_data_source_name'
  required_fields:
    - 'Field1'
    - 'Field2'
    - 'Field3'

# Reference to reference data (if needed)
reference_data:
  New_Reference_Data:
    max_age_days: 90  # Override global setting for this analytic

# Validation configuration
validations:
  - rule: 'existing_validation_rule'  # Use an existing rule from validation_rules.py
    description: 'Description of the validation'
    rationale: 'Why this validation is important'
    parameters:
      param1: 'value1'
      param2: 'value2'
  
  # You can add multiple validation rules
  - rule: 'another_validation_rule'
    description: 'Another validation description'
    parameters:
      paramA: 'valueA'

# Error thresholds
thresholds:
  error_percentage: 5.0  # Maximum acceptable error rate
  rationale: 'Business justification for this threshold'

# Reporting configuration
reporting:
  group_by: 'GroupField'  # Field to group results by
  summary_fields: ['GC', 'PC', 'DNC', 'Total', 'DNC_Percentage']
  detail_required: True

# Additional metadata
report_metadata:
  owner: 'Responsible Team'
  review_frequency: 'Monthly'
  last_revised: '2025-05-10'
  version: '1.0'
  contact_email: 'team@example.com'
```

## Step 5: Add Custom Validation Rules (If Needed)

If your analytic needs a new validation rule, add it to `validation_rules.py`:

```python
# In validation_rules.py
@staticmethod
def new_validation_rule(df: pd.DataFrame, params: Dict) -> pd.Series:
    """
    Description of what this validation rule checks

    Args:
        df: DataFrame containing the data
        params: Dict with parameters needed for this validation

    Returns:
        Series with True for rows that conform, False for non-conforming
    """
    # Extract parameters
    param1 = params.get('param1')
    param2 = params.get('param2')
    
    if not param1 or not param2:
        logger.error("Missing required parameters for new_validation_rule")
        return pd.Series(False, index=df.index)
    
    # Initialize result as all True
    result = pd.Series(True, index=df.index)
    
    # Implement your validation logic
    # For example:
    for idx, row in df.iterrows():
        if some_condition:
            result[idx] = False
    
    return result
```

## Step 6: Test Your New Analytic

1. **Create test data** that includes both conforming and non-conforming records
2. **Run the analytic** using the GUI or command line:

```bash
# Command line
python enhanced_qa_analytics.py -a XX -s test_data/test_file.xlsx -o output
```

3. **Review the results** and verify that:
   - Data is loaded correctly
   - Validations identify issues correctly
   - Reports show proper grouping and summary statistics

## Step 7: Document Your Analytic

Add documentation for your new analytic:

1. **Update any user guides** to include the new analytic
2. **Document validation rules** and their parameters
3. **Create sample test data** for future testing
4. **Document any special considerations** or business rules

## Common Issues and Troubleshooting

### Data Loading Issues

If data doesn't load properly:
- Check that column names match expected values or aliases
- Verify that required columns exist in your data source
- Check the file format matches what's expected

### Validation Rule Issues

If validations don't work correctly:
- Verify parameters are passed correctly from your config file
- Check for any data type conversion issues (dates, numbers, etc.)
- Add debug logging to see intermediate validation results

### Reference Data Issues

If reference data isn't being used correctly:
- Verify the reference data file exists and is properly formatted
- Check that key fields exist in both your data and reference data
- Ensure the reference data is being loaded before validations run

## Best Practices

1. **Start by copying** a similar existing analytic configuration
2. **Use descriptive names** for your validations and parameters
3. **Test with small datasets** before processing large files
4. **Add comments** in your configuration for complex validations
5. **Group related analytics** in the same data source when possible
6. **Reuse validation rules** instead of creating duplicates
7. **Version your configurations** in source control

## Example: Adding a New Workpaper Quality Check Analytic

Here's a complete example adding a new analytic for workpaper quality checks:

```yaml
# configs/qa_04.yaml
analytic_id: 04
analytic_name: 'Workpaper Evidence Quality'
analytic_description: 'Validates that workpapers have sufficient evidence documentation attached.'

data_source:
  name: 'audit_workpaper_approvals'
  required_fields:
    - 'Audit TW ID'
    - 'TW submitter'
    - 'Evidence Count'
    - 'Complexity Rating'

validations:
  - rule: 'evidence_sufficiency'
    description: 'Ensures sufficient evidence based on complexity'
    rationale: 'Higher complexity workpapers require more supporting evidence.'
    parameters:
      evidence_field: 'Evidence Count'
      complexity_field: 'Complexity Rating'
      requirements:
        'High': 5
        'Medium': 3
        'Low': 1

thresholds:
  error_percentage: 3.0
  rationale: 'Evidence is critical for audit quality and compliance.'

reporting:
  group_by: 'TW submitter'
  summary_fields: ['GC', 'PC', 'DNC', 'Total', 'DNC_Percentage']
  detail_required: True

report_metadata:
  owner: 'Quality Assurance Team'
  review_frequency: 'Monthly'
  last_revised: '2025-05-10'
  version: '1.0'
```

And the corresponding validation rule:

```python
# In validation_rules.py
@staticmethod
def evidence_sufficiency(df: pd.DataFrame, params: Dict) -> pd.Series:
    """
    Validates that sufficient evidence is attached based on complexity
    
    Args:
        df: DataFrame containing the data
        params: Dict with evidence_field, complexity_field, and requirements
        
    Returns:
        Series with True for rows with sufficient evidence
    """
    evidence_field = params.get('evidence_field')
    complexity_field = params.get('complexity_field')
    requirements = params.get('requirements', {})
    
    if not evidence_field or not complexity_field or not requirements:
        logger.error("Missing required parameters for evidence_sufficiency")
        return pd.Series(False, index=df.index)
    
    result = pd.Series(False, index=df.index)
    
    for idx, row in df.iterrows():
        complexity = row[complexity_field]
        evidence_count = row[evidence_field]
        
        # Skip if we have null values
        if pd.isna(complexity) or pd.isna(evidence_count):
            continue
            
        # Get required evidence count for this complexity
        required_count = requirements.get(complexity, 0)
        
        # Record passes if it has at least the required evidence count
        if evidence_count >= required_count:
            result[idx] = True
            
    return result
```

## Implementing Multiple Analytics Efficiently

When implementing multiple related analytics, consider these strategies to maximize efficiency:

### Using Template Files

Create template YAML files for common analytic types:

```
templates/
  ├── approval_validation_template.yaml
  ├── risk_assessment_template.yaml
  └── control_testing_template.yaml
```

Copy and modify these templates when creating new analytics of similar types.

### Batch Creation Script

For large-scale implementation, create a simple script to generate multiple analytics at once:

```python
import yaml
import os

def create_analytics_batch(template_path, analytics_data, output_dir="configs"):
    """Create multiple analytics configs from a template and data list"""
    with open(template_path, 'r') as f:
        template = yaml.safe_load(f)
    
    for analytic in analytics_data:
        # Create a copy of the template
        config = template.copy()
        
        # Update with analytic-specific data
        config['analytic_id'] = analytic['id']
        config['analytic_name'] = analytic['name']
        config['analytic_description'] = analytic['description']
        
        # Update other fields as needed
        if 'validations' in analytic:
            config['validations'] = analytic['validations']
        
        # Write to file
        output_path = os.path.join(output_dir, f"qa_{analytic['id']}.yaml")
        with open(output_path, 'w') as f:
            yaml.dump(config, f, default_flow_style=False)
        
        print(f"Created analytic QA-{analytic['id']}: {analytic['name']}")

# Example usage
analytics = [
    {'id': '05', 'name': 'Control Design Assessment', 'description': 'Validates control design quality'},
    {'id': '06', 'name': 'Control Effectiveness', 'description': 'Validates control operating effectiveness'},
    # Add more analytics...
]

create_analytics_batch('templates/control_testing_template.yaml', analytics)
```

## Maintaining Your Analytics Framework

As your analytics framework grows, consider these maintenance best practices:

### Version Control

Track all configuration changes in a version control system:
- Record who made changes and why
- Maintain a history of analytics evolution
- Support rollback to previous versions if needed

### Documentation Repository

Maintain a central documentation repository:
- Catalog all available analytics
- Document validation rules and their purposes
- Include example data sets and expected results
- Record common issues and solutions

### Regular Reviews

Schedule periodic reviews of your analytics:
- Verify thresholds are still appropriate
- Update analytics for changing business requirements
- Remove obsolete analytics
- Consolidate similar analytics

By following this comprehensive guide, you can confidently add new analytics to your Enhanced QA Analytics Framework in a structured and maintainable way, ensuring consistency and quality across your entire quality assurance program.
