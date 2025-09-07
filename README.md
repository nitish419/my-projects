import pandas as pd
import numpy as np

def load_data(filepath):
    """
    Loads sales data from a CSV file.

    Args:
        filepath (str): The path to the CSV file.

    Returns:
        pandas.DataFrame: The loaded DataFrame, or None if an error occurs.
    """
    try:
       
        df = pd.read_csv(filepath)
        print(f"Successfully loaded data from {filepath}")
        return df
    except FileNotFoundError:
        print(f"Error: The file at {filepath} was not found.")
        return None
    except Exception as e:
        print(f"An unexpected error occurred while loading the file: {e}")
        return None

def clean_data(df):
    """
    Cleans the sales data.

    This function performs the following cleaning steps:
    1. Removes duplicate rows.
    2. Handles missing values by dropping rows with any missing data.
    3. Converts data types for 'Date' (to datetime) and 'Price' (to numeric).

    Args:
        df (pandas.DataFrame): The raw DataFrame.

    Returns:
        pandas.DataFrame: The cleaned DataFrame.
    """
    if df is None:
        return None

    print("\nStarting data cleaning process...")

   
    initial_rows = len(df)
    df.drop_duplicates(inplace=True)
    rows_after_duplicates = len(df)
    print(f"- Removed {initial_rows - rows_after_duplicates} duplicate rows.")

    df.dropna(inplace=True)
    rows_after_missing = len(df)
    print(f"- Removed {rows_after_duplicates - rows_after_missing} rows with missing values.")

   
    df['Date'] = pd.to_datetime(df['Date'], errors='coerce')
    

    df['Price'] = pd.to_numeric(df['Price'], errors='coerce')
    df['Quantity'] = pd.to_numeric(df['Quantity'], errors='coerce')


    df.dropna(subset=['Date', 'Price', 'Quantity'], inplace=True)
    
    print("\nData cleaning complete.")
    return df

def generate_summary_report(df):
    """
    Generates a summary report of the sales data.

    The report includes:
    - Total revenue
    - Average transaction value
    - Most popular product
    - Total number of transactions

    Args:
        df (pandas.DataFrame): The cleaned DataFrame.

    Returns:
        pandas.DataFrame: A DataFrame containing the summary statistics.
    """
    if df is None or df.empty:
        print("Cannot generate report: DataFrame is empty or invalid.")
        return pd.DataFrame()

    print("\nGenerating summary report...")

    df['Total_Sales'] = df['Price'] * df['Quantity']
    total_revenue = df['Total_Sales'].sum()
    avg_transaction_value = df['Total_Sales'].mean()
    total_transactions = df.shape[0]

    most_popular_product = df.groupby('Product')['Quantity'].sum().idxmax()


    summary = pd.DataFrame({
        'Metric': ['Total Revenue', 'Average Transaction Value', 'Most Popular Product', 'Total Transactions'],
        'Value': [
            f"${total_revenue:,.2f}",
            f"${avg_transaction_value:,.2f}",
            most_popular_product,
            total_transactions
        ]
    })
    
    return summary

def generate_sales_by_category_report(df):
    """
    Generates a report of total sales grouped by product category.

    Args:
        df (pandas.DataFrame): The cleaned DataFrame.

    Returns:
        pandas.DataFrame: A DataFrame with total sales per category.
    """
    if df is None or df.empty:
        print("Cannot generate sales by category report: DataFrame is empty or invalid.")
        return pd.DataFrame()

    print("\nGenerating sales report by category...")
    

    sales_by_category = df.groupby('Category')['Total_Sales'].sum().reset_index()
    sales_by_category.rename(columns={'Total_Sales': 'Total Revenue'}, inplace=True)
    

    sales_by_category.sort_values(by='Total Revenue', ascending=False, inplace=True)
    
    return sales_by_category
    
def generate_sales_by_user_report(df):
    """
    Generates a report of total sales grouped by customer ID.
    
    Args:
        df (pandas.DataFrame): The cleaned DataFrame.

    Returns:
        pandas.DataFrame: A DataFrame with total sales per customer ID.
    """
    if df is None or df.empty:
        print("Cannot generate sales by user report: DataFrame is empty or invalid.")
        return pd.DataFrame()
    
    print("\nGenerating sales report by user...")
    
    sales_by_user = df.groupby('Customer_ID')['Total_Sales'].sum().reset_index()
    sales_by_user.rename(columns={'Total_Sales': 'Total Revenue'}, inplace=True)

    sales_by_user.sort_values(by='Total Revenue', ascending=False, inplace=True)
    
    return sales_by_user

def generate_sales_for_user(df, user_id):
    """
    Generates a sales report for a specific user.

    Args:
        df (pandas.DataFrame): The cleaned DataFrame.
        user_id (int): The ID of the customer to report on.

    Returns:
        pandas.DataFrame: A DataFrame with the sales data for the specified user,
                          or an empty DataFrame if the user is not found.
    """
    if df is None or df.empty:
        print("Cannot generate report: DataFrame is empty or invalid.")
        return pd.DataFrame()

    user_df = df[df['Customer_ID'] == user_id]

    if user_df.empty:
        print(f"\nNo sales data found for Customer ID: {user_id}")
        return pd.DataFrame()

    print(f"\nGenerating sales report for Customer ID: {user_id}...")

   
    user_sales_summary = pd.DataFrame({
        'Metric': ['Total Revenue', 'Total Transactions', 'Total Products'],
        'Value': [
            f"${user_df['Total_Sales'].sum():,.2f}",
            user_df.shape[0],
            user_df['Quantity'].sum()
        ]
    })
    
    return user_sales_summary


if __name__ == "__main__":
   
    data = {
        'Date': ['2023-01-01', '2023-01-02', '2023-01-03', '2023-01-04', '2023-01-05'],
        'Product': ['Laptop', 'Mouse', 'Keyboard', 'Laptop', 'Monitor'],
        'Category': ['Electronics', 'Accessories', 'Accessories', 'Electronics', 'Electronics'],
        'Price': [1200, 25, 75, 1200, 300],
        'Quantity': [1, 5, 2, 1, 1],
        'Customer_ID': [101, 102, 103, 104, 105]
    }
    
   
    print("Simulating data loading from 'sales.csv'...")
    sales_df = pd.DataFrame(data)

    

    if sales_df is not None and not sales_df.empty:
   
        cleaned_df = clean_data(sales_df)
        
       
        summary_report = generate_summary_report(cleaned_df)
        print("\n--- Sales Summary Report ---")
        print(summary_report.to_string(index=False))

  
        category_report = generate_sales_by_category_report(cleaned_df)
        print("\n--- Total Sales by Category ---")
        print(category_report.to_string(index=False))
        
        
        
        user_report = generate_sales_by_user_report(cleaned_df)
        print("\n--- Total Sales by Customer ---")
        print(user_report.to_string(index=False))

 
        try:
            user_input = int(input("\nEnter a Customer ID to view a specific report (e.g., 101): "))
            user_specific_report = generate_sales_for_user(cleaned_df, user_input)
            if not user_specific_report.empty:
                print(f"\n--- Report for Customer ID: {user_input} ---")
                print(user_specific_report.to_string(index=False))
        except ValueError:
            print("\nInvalid input. Please enter a numeric Customer ID.")
