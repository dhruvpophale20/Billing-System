import csv
import time
from collections import Counter
import tkinter as tk
from tkinter import messagebox
from tkinter import ttk

def t():
    current_time = time.localtime()
    f_time = time.strftime("%Y-%m-%d %H:%M:%S")
    return f_time

def write_to_csv(bill_number, bill_counter, Rate_List, total_amount):
    with open('bill_records.csv', mode='a', newline='') as file:
        writer = csv.writer(file)
        writer.writerow(["BILL NO.", "SR NO.", "ITEM", "QUANTITY", "TOTAL PRICE"])
        n = 1
        for item, quantity in bill_counter.items():
            price = Rate_List[item]
            total_price = price * quantity
            writer.writerow([bill_number, n, item, quantity, total_price])
            n += 1
        writer.writerow(["", "", "", "TOTAL AMOUNT", total_amount])

def billing(item_dict, bill_number, Rate_List):
    bill_counter = Counter()
    total_amount = 0
    for item, quantity in item_dict.items():
        if item in Rate_List:
            price = Rate_List[item]
            total_price = quantity * price
            bill_counter[item] = quantity
            total_amount += total_price
    write_to_csv(bill_number, bill_counter, Rate_List, total_amount)
    return bill_counter, total_amount

def generate_bill():
    try:
        no_of_item = int(entry_no_of_item.get())
        item_dict = {}
        for i in range(no_of_item):
            item_name = entry_item_names[i].get()
            item_num = int(entry_item_quantities[i].get())
            if item_name in item_dict:
                item_dict[item_name] += item_num
            else:
                item_dict[item_name] = item_num
        bill_no = entry_bill_no.get()
        Bill, total_amount = billing(item_dict, bill_no, Rate_List)
        text_bill_details.delete(1.0, tk.END)
        text_bill_details.insert(tk.END, f"---:BILL:---\n")
        text_bill_details.insert(tk.END, f"Time: {t()}\n")
        for item, quantity in Bill.items():
            price = Rate_List[item]
            total_price = price * quantity
            text_bill_details.insert(tk.END, f"{item} - {quantity} x {price} = {total_price}\n")
        text_bill_details.insert(tk.END, f"Total Amount: {total_amount}\n")
        messagebox.showinfo("Bill Generated", "Bill has been generated successfully!")
    except Exception as e:
        messagebox.showerror("Error", str(e))

def new_bill():
    entry_bill_no.delete(0, tk.END)
    entry_no_of_item.delete(0, tk.END)
    for entry in entry_item_names:
        entry.destroy()
    for entry in entry_item_quantities:
        entry.destroy()
    entry_item_names.clear()
    entry_item_quantities.clear()
    frame_items.grid_remove()
    text_bill_details.delete(1.0, tk.END)

def center_window(win):
    win.update_idletasks()
    width = win.winfo_width()
    height = win.winfo_height()
    x = (win.winfo_screenwidth() // 2) - (width // 2)
    y = (win.winfo_screenheight() // 2) - (height // 2)
    win.geometry(f'{width}x{height}+{x}+{y}')

# Sample Rate List
Rate_List = {'P1': 30, 'P2': 32, 'P3': 20, 'P4': 20, 'P5': 25, 'P6': 28, 'P7': 18, 'P8': 40, 'P9': 35, 'P10': 15}

# Create GUI Window
root = tk.Tk()
root.title("Billing System")
root.configure(bg='#F0F8FF')

# Frame for Billing Form
frame_billing = tk.Frame(root, bg='#F0F8FF')
frame_billing.grid(row=0, column=0, padx=20, pady=20, sticky='n')

tk.Label(frame_billing, text="Enter Bill Number:", bg='#F0F8FF').grid(row=0, column=0)
entry_bill_no = tk.Entry(frame_billing)
entry_bill_no.grid(row=0, column=1)

tk.Label(frame_billing, text="Enter No. Of Items:", bg='#F0F8FF').grid(row=1, column=0)
entry_no_of_item = tk.Entry(frame_billing)
entry_no_of_item.grid(row=1, column=1)

entry_item_names = []
entry_item_quantities = []

def add_item_fields():
    no_of_item = int(entry_no_of_item.get())
    for widget in frame_items.winfo_children():
        widget.destroy()
    for i in range(no_of_item):
        tk.Label(frame_items, text=f"Item {i+1} Name:", bg='#F0F8FF').grid(row=i, column=0)
        item_name_entry = tk.Entry(frame_items)
        item_name_entry.grid(row=i, column=1)
        entry_item_names.append(item_name_entry)
        tk.Label(frame_items, text=f"Item {i+1} Quantity:", bg='#F0F8FF').grid(row=i, column=2)
        item_quantity_entry = tk.Entry(frame_items)
        item_quantity_entry.grid(row=i, column=3)
        entry_item_quantities.append(item_quantity_entry)
    frame_items.grid()

button_add_items = tk.Button(frame_billing, text="Add Item Fields", command=add_item_fields, bg='#4682B4', fg='white')
button_add_items.grid(row=2, column=0, columnspan=2, pady=5)

frame_items = tk.Frame(frame_billing, bg='#F0F8FF')
frame_items.grid(row=3, column=0, columnspan=2)
frame_items.grid_remove()

button_generate_bill = tk.Button(frame_billing, text="Generate Bill", command=generate_bill, bg='#4682B4', fg='white')
button_generate_bill.grid(row=4, column=0, columnspan=2, pady=5)

button_new_bill = tk.Button(frame_billing, text="New Bill", command=new_bill, bg='#4682B4', fg='white')
button_new_bill.grid(row=5, column=0, columnspan=2, pady=5)

button_exit = tk.Button(frame_billing, text="Exit", command=root.quit, bg='#4682B4', fg='white')
button_exit.grid(row=6, column=0, columnspan=2, pady=5)

text_bill_details = tk.Text(frame_billing, width=60, height=20, bg='#F0F8FF')
text_bill_details.grid(row=7, column=0, columnspan=2, pady=5)

# Frame for Rate List
frame_rate_list = tk.Frame(root, bg='#F0F8FF')
frame_rate_list.grid(row=0, column=1, padx=20, pady=20, sticky='n')

tk.Label(frame_rate_list, text="Rate List", bg='#F0F8FF', font=('Arial', 14, 'bold')).grid(row=0, column=0, columnspan=2)
tree = ttk.Treeview(frame_rate_list, columns=('Item', 'Price'), show='headings')
tree.heading('Item', text='Item')
tree.heading('Price', text='Price')
for item, price in Rate_List.items():
    tree.insert('', tk.END, values=(item, price))
tree.grid(row=1, column=0, columnspan=2)

# Center the window
center_window(root)

root.mainloop()
