import mysql.connector
import tkinter as tk
from tkinter import messagebox, font, ttk 

DB_NAME = "INDIAN_AUDIT_DB"
DB_USER = "root"
DB_PASS = "Saxmachine4lyf@123" 

def connect_server_only():
    try:
        db = mysql.connector.connect(host="localhost", user=DB_USER, password=DB_PASS)
        return db
    except mysql.connector.Error as err:

        if err.errno == 1045:
            messagebox.showerror("Connection Error", f"Access Denied (Error 1045). Please check your password in the DB_PASS variable.")
        else:
            messagebox.showerror("Connection Error", f"Cannot connect to MySQL Server. Error: {err}")
        raise

def connect_db():
    try:
        db = mysql.connector.connect(host="localhost", user=DB_USER, password=DB_PASS, database=DB_NAME)
        return db
    except mysql.connector.Error as err:
        if err.errno == 1045:
            messagebox.showerror("Connection Error", f"Access Denied (Error 1045). Please check your password in the DB_PASS variable.")
        elif err.errno == 1049:
             messagebox.showerror("Connection Error", f"Database Not Found (Error 1049). Ensure MySQL is running and DB setup was successful.")
        else:
            messagebox.showerror("Connection Error", f"Cannot connect to {DB_NAME}. Error: {err}")
        raise

def setup_database():
    try:
        db_server = connect_server_only()
        cursor = db_server.cursor()
        cursor.execute(f"CREATE DATABASE IF NOT EXISTS {DB_NAME}")
        cursor.close()
        db_server.close()
        
        db = connect_db()
        cursor = db.cursor()
        
        cursor.execute("""
        CREATE TABLE IF NOT EXISTS Companies (
            CompanyID INT AUTO_INCREMENT PRIMARY KEY,
            CompanyName VARCHAR(255) NOT NULL,
            CIN VARCHAR(50) UNIQUE NULL)
        """)
        
        cursor.execute("""
        CREATE TABLE IF NOT EXISTS Financials (
            FinID INT AUTO_INCREMENT PRIMARY KEY,
            CompanyID INT,
            RevenueFromOps INT,
            OtherIncome INT,
            Purchases INT,
            EmployeeBenefitExp INT,
            FinanceCost INT,
            DepreciationAndAmort INT,
            OtherExpenses INT,
            CurrentAssets INT,
            NonCurrentAssets INT,
            CurrentLiabilities INT,
            NonCurrentLiabilities INT,
            ShareCapital INT,
            ReservesAndSurplus INT,
            TaxRate INT,
            PBT FLOAT,
            PAT FLOAT,
            ReturnOnEquity FLOAT,
            DebtToEquityRatio FLOAT,
            CurrentRatio FLOAT,
            FOREIGN KEY (CompanyID) REFERENCES Companies(CompanyID))
        """)
        db.commit()
        cursor.close()
        db.close()
    except Exception as e:

        if 'Access Denied' in str(e): return 
        messagebox.showerror("Initial Setup Error", f"A fatal error occurred during database setup: {e}")

def insert_data(co_name, cin, rev_ops, other_inc, pur, emp_bene, fin_cost, dep_amor, other_exp, curr_assets, non_curr_assets, curr_liab, non_curr_liab, share_cap, res_sur, tax_rate):
    try:
        db = connect_db()
        cursor = db.cursor()
        
        cursor.execute("INSERT INTO Companies (CompanyName, CIN) VALUES (%s, %s)", (co_name, cin))
        co_id = cursor.lastrowid
        
        total_inc = rev_ops + other_inc
        total_exp_excl_tax = pur + emp_bene + fin_cost + dep_amor + other_exp
        pbt = total_inc - total_exp_excl_tax
        
        tax_amt = (tax_rate * pbt) / 100
        pat = pbt - tax_amt

        share_funds = share_cap + res_sur + pat 

        total_debt = fin_cost + non_curr_liab + curr_liab 
        roe = (pat / share_funds) * 100 if share_funds > 0 else float('inf')
        d_to_e_ratio = total_debt / share_funds if share_funds != 0 else float('inf')
        curr_ratio = curr_assets / curr_liab if curr_liab > 0 else float('inf')

        cursor.execute("""
        INSERT INTO Financials (
            CompanyID, RevenueFromOps, OtherIncome, Purchases, EmployeeBenefitExp, FinanceCost, 
            DepreciationAndAmort, OtherExpenses, CurrentAssets, NonCurrentAssets, CurrentLiabilities, 
            NonCurrentLiabilities, ShareCapital, ReservesAndSurplus, TaxRate, PBT, PAT, ReturnOnEquity, 
            DebtToEquityRatio, CurrentRatio) 
        VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)""",
        (co_id, rev_ops, other_inc, pur, emp_bene, fin_cost, dep_amor, other_exp, 
         curr_assets, non_curr_assets, curr_liab, non_curr_liab, share_cap, res_sur, tax_rate, 
         pbt, pat, roe, d_to_e_ratio, curr_ratio))
        
        db.commit()
        cursor.close()
        db.close()
        messagebox.showinfo("Success", "Financial data inserted successfully!")
    except mysql.connector.Error as err:
        messagebox.showerror("Database Error", f"An error occurred during insert: {err}")
    except Exception as e:
        messagebox.showerror("Calculation Error", f"An internal error occurred: {e}")

def fetch_company_details(co_id):
    try:
        db = connect_db()
        cursor = db.cursor()
        
        cursor.execute("SELECT * FROM Companies WHERE CompanyID = %s", (co_id,))
        company = cursor.fetchone()
        
        cursor.execute("SELECT * FROM Financials WHERE CompanyID = %s", (co_id,))
        financials = cursor.fetchone()
        
        cursor.close()
        db.close()
        
        if company and financials:
            
            pbt_display = f"Rs. {financials[16]:,.2f}"
            pat_display = f"Rs. {financials[17]:,.2f}"
            roe_display = f"{financials[18]:.2f}%"
            debt_equity_display = f"{financials[19]:.2f}" if financials[19] != float('inf') else "N/A (No Equity)"
            current_ratio_display = f"{financials[20]:.2f}" if financials[20] != float('inf') else "N/A (No Current Liabilities)"
            
            details = f"Company ID: {company[0]}\nCompany Name: {company[1]}\nCIN: {company[2]}\n\n"
            details += f"--- Financial Data ---\n"
            details += f"Revenue from Operations: {financials[2]:,}\n"
            details += f"Other Income: {financials[3]:,}\n"
            details += f"Total Expenses (Excl Tax): {financials[4] + financials[5] + financials[6] + financials[7] + financials[8]:,}\n"
            details += f"Share Capital: {financials[13]:,}\n"
            details += f"Reserves and Surplus: {financials[14]:,}\n"
            details += f"Tax Rate: {financials[15]}%\n\n"
            
            details += f"--- Key Ratios (Indian Context) ---\n"
            details += f"Profit Before Tax (PBT): {pbt_display}\n"
            details += f"Profit After Tax (PAT): {pat_display}\n"
            details += f"Return on Equity (ROE): {roe_display}\n"
            details += f"Debt-to-Equity Ratio: {debt_equity_display}\n"
            details += f"Current Ratio (Liquidity): {current_ratio_display}\n"
            
            messagebox.showinfo("Company Details", details)
        else:
            messagebox.showwarning("Not Found", "Company not found!")
            
    except mysql.connector.Error as err:
        messagebox.showerror("Database Error", f"An error occurred during fetch: {err}")

def update_company(co_id, new_name, cin, rev_ops, other_inc, pur, emp_bene, fin_cost, dep_amor, other_exp, curr_assets, non_curr_assets, curr_liab, non_curr_liab, share_cap, res_sur, tax_rate):
    try:
        db = connect_db()
        cursor = db.cursor()
        
        cursor.execute("UPDATE Companies SET CompanyName = %s, CIN = %s WHERE CompanyID = %s", (new_name, cin, co_id))

        total_inc = rev_ops + other_inc
        total_exp_excl_tax = pur + emp_bene + fin_cost + dep_amor + other_exp
        pbt = total_inc - total_exp_excl_tax
        
        tax_amt = (tax_rate * pbt) / 100
        pat = pbt - tax_amt

        share_funds = share_cap + res_sur + pat 

        total_debt = fin_cost + non_curr_liab + curr_liab 
        roe = (pat / share_funds) * 100 if share_funds > 0 else float('inf')
        d_to_e_ratio = total_debt / share_funds if share_funds != 0 else float('inf')
        curr_ratio = curr_assets / curr_liab if curr_liab > 0 else float('inf')

        cursor.execute("""
        UPDATE Financials SET RevenueFromOps=%s, OtherIncome=%s, Purchases=%s, EmployeeBenefitExp=%s, 
        FinanceCost=%s, DepreciationAndAmort=%s, OtherExpenses=%s, CurrentAssets=%s, NonCurrentAssets=%s, 
        CurrentLiabilities=%s, NonCurrentLiabilities=%s, ShareCapital=%s, ReservesAndSurplus=%s, TaxRate=%s, 
        PBT=%s, PAT=%s, ReturnOnEquity=%s, DebtToEquityRatio=%s, CurrentRatio=%s
        WHERE CompanyID=%s""",
        (rev_ops, other_inc, pur, emp_bene, fin_cost, dep_amor, other_exp, 
         curr_assets, non_curr_assets, curr_liab, non_curr_liab, share_cap, res_sur, tax_rate, 
         pbt, pat, roe, d_to_e_ratio, curr_ratio, co_id))

        db.commit()
        cursor.close()
        db.close()
        messagebox.showinfo("Success", "Company updated successfully!")
    except mysql.connector.Error as err:
        messagebox.showerror("Database Error", f"An error occurred during update: {err}")
    except Exception as e:
        messagebox.showerror("Calculation Error", f"An internal error occurred: {e}")

def delete_company(co_id):
    try:
        db = connect_db()
        cursor = db.cursor()
        cursor.execute("DELETE FROM Financials WHERE CompanyID = %s", (co_id,))
        cursor.execute("DELETE FROM Companies WHERE CompanyID = %s", (co_id,))
        db.commit()
        cursor.close()
        db.close()
        messagebox.showinfo("Success", "Company deleted successfully!")
    except mysql.connector.Error as err:
        messagebox.showerror("Database Error", f"An error occurred during delete: {err}")

def validate_and_submit():
    try:
        co_name = company_name_entry.get()
        cin = cin_entry.get()
        rev_ops = int(revenue_ops_entry.get())
        other_inc = int(other_income_entry.get())
        pur = int(purchases_entry.get())
        emp_bene = int(emp_exp_entry.get())
        fin_cost = int(finance_cost_entry.get())
        dep_amor = int(deprec_amort_entry.get())
        other_exp = int(other_exp_entry.get())
        curr_assets = int(curr_assets_entry.get())
        non_curr_assets = int(non_curr_assets_entry.get())
        curr_liab = int(curr_liab_entry.get())
        non_curr_liab = int(non_curr_liab_entry.get())
        share_cap = int(share_capital_entry.get())
        res_sur = int(reserves_surplus_entry.get())
        tax_rate = int(tax_rate_entry.get())

        if not co_name.strip():
            messagebox.showerror("Input Error", "Company Name cannot be empty.")
            return
        if not cin.strip():
            messagebox.showerror("Input Error", "CIN cannot be empty.")
            return
        if tax_rate < 0 or tax_rate > 100:
            messagebox.showerror("Input Error", "Tax Rate must be between 0 and 100.")
            return
        
        if any(x < 0 for x in [rev_ops, other_inc, pur, emp_bene, fin_cost, dep_amor, other_exp, curr_assets, non_curr_assets, curr_liab, non_curr_liab, share_cap, res_sur]):
             messagebox.showerror("Input Error", "All monetary inputs must be non-negative.")
             return

        insert_data(co_name, cin, rev_ops, other_inc, pur, emp_bene, fin_cost, dep_amor, other_exp, curr_assets, non_curr_assets, curr_liab, non_curr_liab, share_cap, res_sur, tax_rate)
    except ValueError:
        messagebox.showerror("Input Error", "Please enter valid integers for all financial fields.")
    except Exception as e:
        messagebox.showerror("System Error", f"An unexpected error occurred: {e}")

def print_all_companies():
    try:
        db = connect_db()
        cursor = db.cursor()
        cursor.execute("SELECT CompanyID, CompanyName, CIN FROM Companies")
        companies = cursor.fetchall()
        cursor.close()
        db.close()
        
        details = "ID | Company Name | CIN\n---------------------------------------\n"
        for c in companies:
            details += f"{c[0]:<2} | {c[1]:<15} | {c[2]}\n"
        
        if len(companies) == 0:
            messagebox.showinfo("All Companies", "No companies found in the database.")
        else:
            messagebox.showinfo("All Companies", details.strip())
            
    except mysql.connector.Error as err:
        messagebox.showerror("Database Error", f"Could not fetch companies: {err}")

def execute_update_delete(action):
    try:
        co_id = int(company_id_entry.get())
        if action == 'delete':
            delete_company(co_id)
            return
        
        new_name = company_name_entry.get()
        cin = cin_entry.get()
        rev_ops = int(revenue_ops_entry.get())
        other_inc = int(other_income_entry.get())
        pur = int(purchases_entry.get())
        emp_bene = int(emp_exp_entry.get())
        fin_cost = int(finance_cost_entry.get())
        dep_amor = int(deprec_amort_entry.get())
        other_exp = int(other_exp_entry.get())
        curr_assets = int(curr_assets_entry.get())
        non_curr_assets = int(non_curr_assets_entry.get())
        curr_liab = int(curr_liab_entry.get())
        non_curr_liab = int(non_curr_liab_entry.get())
        share_cap = int(share_capital_entry.get())
        res_sur = int(reserves_surplus_entry.get())
        tax_rate = int(tax_rate_entry.get())

        update_company(co_id, new_name, cin, rev_ops, other_inc, pur, emp_bene, fin_cost, dep_amor, other_exp, curr_assets, non_curr_assets, curr_liab, non_curr_liab, share_cap, res_sur, tax_rate)
        
    except ValueError:
        messagebox.showerror("Input Error", "Please enter a valid integer for Company ID and all financial fields.")
    except Exception as e:
        messagebox.showerror("System Error", f"An unexpected error occurred: {e}")


root = tk.Tk()
root.title(f"Indian Company Audit System ({DB_NAME})")

root.geometry("700x550") 
root.configure(bg="#f0f0f0") 

title_font = font.Font(family="Georgia", size=22, weight="bold")
title_label = tk.Label(root, text="🇮🇳 Company Financial Audit (Simplified)", font=title_font, bg="#f0f0f0", fg="#003366") # Navy Blue
title_label.pack(pady=15)


main_frame = tk.Frame(root)
main_frame.pack(fill=tk.BOTH, expand=True, padx=25, pady=5) 

canvas = tk.Canvas(main_frame, bg="#f8f8f8", borderwidth=1, relief=tk.SUNKEN) 
canvas.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)

scrollbar = ttk.Scrollbar(main_frame, orient=tk.VERTICAL, command=canvas.yview)
scrollbar.pack(side=tk.RIGHT, fill=tk.Y)

canvas.configure(yscrollcommand=scrollbar.set)
canvas.bind('<Configure>', lambda e: canvas.configure(scrollregion = canvas.bbox("all")))


input_frame = tk.Frame(canvas, bg="#f8f8f8") 


canvas.create_window((0, 0), window=input_frame, anchor="nw")



labels_data = [
    ("Company Name:", "text"),
    ("CIN (Corporate ID No.):", "text"),
    ("--- P&L (in Rs.) ---", "separator"),
    ("Revenue from Operations:", "int"),
    ("Other Income:", "int"),
    ("Purchases of Stock-in-Trade:", "int"),
    ("Employee Benefit Expenses:", "int"),
    ("Finance Cost (Interest Paid):", "int"),
    ("Depreciation and Amort.:", "int"),
    ("Other Expenses:", "int"),
    ("--- Balance Sheet (in Rs.) ---", "separator"),
    ("Current Assets:", "int"),
    ("Non-Current Assets:", "int"),
    ("Current Liabilities (Short-Term):", "int"),
    ("Non-Current Liabilities (Long-Term):", "int"),
    ("Share Capital:", "int"),
    ("Reserves & Surplus (Previous Year):", "int"),
    ("--- Tax & Other ---", "separator"),
    ("Applicable Tax Rate (%):", "int"),
]

entries = {}
row_num = 0
for label_text, type in labels_data:
    if type == "separator":
        sep_label = tk.Label(input_frame, text=label_text, bg="#f8f8f8", fg="#6A5ACD", font=("Arial", 11, "bold"))
        sep_label.grid(row=row_num, column=0, columnspan=2, pady=(12, 6), sticky=tk.W)
        row_num += 1
        continue
    
    label = tk.Label(input_frame, text=label_text, bg="#f8f8f8", fg="#333333", font=("Arial", 11))
    label.grid(row=row_num, column=0, padx=10, pady=3, sticky=tk.W)
    

    temp_name = label_text.lower()
    for char in [":", ".", "-", "(", ")", "&", "%"]:
        temp_name = temp_name.replace(char, " ")
    entry_name = "_".join(temp_name.split()).strip("_")

    entry = tk.Entry(input_frame, width=35, font=("Arial", 11), bd=1, relief=tk.FLAT, bg="white")
    entry.grid(row=row_num, column=1, padx=10, pady=3)
    entries[entry_name] = entry
    row_num += 1


input_frame.update_idletasks()
canvas.config(scrollregion=canvas.bbox("all"))


company_name_entry = entries['company_name']
cin_entry = entries['cin_corporate_id_no']
revenue_ops_entry = entries['revenue_from_operations']
other_income_entry = entries['other_income']
purchases_entry = entries['purchases_of_stock_in_trade'] 
emp_exp_entry = entries['employee_benefit_expenses']
finance_cost_entry = entries['finance_cost_interest_paid']
deprec_amort_entry = entries['depreciation_and_amort']
other_exp_entry = entries['other_expenses']
curr_assets_entry = entries['current_assets']
non_curr_assets_entry = entries['non_current_assets']
curr_liab_entry = entries['current_liabilities_short_term']
non_curr_liab_entry = entries['non_current_liabilities_long_term']
share_capital_entry = entries['share_capital']
reserves_surplus_entry = entries['reserves_surplus_previous_year']
tax_rate_entry = entries['applicable_tax_rate']



button_frame = tk.Frame(root, bg="#f0f0f0")
button_frame.pack(pady=15)


btn_width = 25
btn_font = ("Arial", 11, "bold")


submit_button = tk.Button(button_frame, text="1. Submit New Data", command=validate_and_submit, bg="#4CAF50", fg="black", width=btn_width, font=btn_font, relief=tk.RAISED, bd=3)
submit_button.grid(row=0, column=0, padx=10, pady=5)

print_all_button = tk.Button(button_frame, text="2. Print All Companies List", command=print_all_companies, bg="#2196F3", fg="black", width=btn_width, font=btn_font, relief=tk.RAISED, bd=3)
print_all_button.grid(row=0, column=1, padx=10, pady=5)

tk.Label(button_frame, text="--- Lookup & Management ---", bg="#f0f0f0", fg="#003366", font=("Arial", 10, "bold")).grid(row=1, columnspan=3, pady=(10, 5))

company_id_label = tk.Label(button_frame, text="Target ID:", bg="#f0f0f0", fg="#333", font=("Arial", 11))
company_id_label.grid(row=2, column=0, padx=5, pady=5, sticky=tk.E)
company_id_entry = tk.Entry(button_frame, width=10, font=("Arial", 11), bd=1, relief=tk.SUNKEN)
company_id_entry.grid(row=2, column=1, padx=5, pady=5, sticky=tk.W)


print_details_button = tk.Button(button_frame, text="3. Print Full Details", command=lambda: fetch_company_details(int(company_id_entry.get())) if company_id_entry.get().isdigit() else messagebox.showerror("Input Error", "Enter valid ID"), bg="#FFC107", fg="black", width=btn_width, font=btn_font, relief=tk.RAISED, bd=3)
print_details_button.grid(row=3, column=0, padx=10, pady=5)

update_button = tk.Button(button_frame, text="4. Update Data (Uses Current Inputs)", command=lambda: execute_update_delete('update'), bg="#FF8C00", fg="black", width=btn_width, font=btn_font, relief=tk.RAISED, bd=3)
update_button.grid(row=3, column=1, padx=10, pady=5)

delete_button = tk.Button(button_frame, text="5. Delete Company", command=lambda: execute_update_delete('delete'), bg="#DC143C", fg="black", width=btn_width, font=btn_font, relief=tk.RAISED, bd=3)
delete_button.grid(row=4, column=0, padx=10, pady=5)

setup_database()

root.mainloop()
