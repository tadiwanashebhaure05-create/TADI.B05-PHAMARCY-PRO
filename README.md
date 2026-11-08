# TADI.B05-PHAMARCY-PRO

 import json
import os
from datetime import datetime

DATA_FILE = "pharmacy_data.json"

def load_data():
    if not os.path.exists(DATA_FILE):
        return {
            "patients": [],
            "inventory": []
        }
    try:
        with open(DATA_FILE, "r") as file:
            data = json.load(file)
            if "patients" not in data:
                data["patients"] = []
            if "inventory" not in data:
                data["inventory"] = []
            return data
    except json.JSONDecodeError:
        return {
            "patients": [],
            "inventory": []
        }

def save_data(data):
    with open(DATA_FILE, "w") as file:
        json.dump(data, file, indent=4)

def generate_patient_id(data):
    if not data["patients"]:
        return "P1001"
    last_id = data["patients"][-1]["patient_id"]
    number = int(last_id[1:]) + 1
    return f"P{number}"

def find_patient(data, patient_id):
    for patient in data["patients"]:
        if patient["patient_id"] == patient_id:
            return patient
    return None

def add_patient(data):
    name = input("Enter patient name: ").strip()
    phone = input("Enter patient phone: ").strip()
    age = input("Enter patient age: ").strip()
    if not name or not phone or not age:
        print("All fields are required.")
        return

    patient_id = generate_patient_id(data)
    patient = {
        "patient_id": patient_id,
        "name": name,
        "phone": phone,
        "age": int(age),
        "prescriptions": []
    }
    data["patients"].append(patient)
    save_data(data)
    print(f"Patient {name} registered successfully. ID: {patient_id}")

def add_medicine(data):
    name = input("Enter medicine name: ").strip()
    qty = input("Enter quantity: ").strip()
    price = input("Enter price per unit: ").strip()

    if not name or not qty or not price:
        print("Medicine name, quantity and price are required.")
        return

    for med in data["inventory"]:
        if med["name"].lower() == name.lower():
            med["quantity"] += int(qty)
            med["price"] = float(price)
            save_data(data)
            print(f"Stock updated for {name}.")
            return

    data["inventory"].append({
        "name": name,
        "quantity": int(qty),
        "price": float(price)
    })
    save_data(data)
    print(f"Medicine {name} added to stock.")

def view_stock(data):
    if not data["inventory"]:
        print("No medicines in stock.")
        return

    print("\nInventory:")
    for med in data["inventory"]:
        print(f"- {med['name']}: {med['quantity']} units, Price: ${med['price']:.2f}")

def search_patient(data):
    patient_id = input("Enter patient ID: ").strip()
    patient = find_patient(data, patient_id)
    if not patient:
        print("Patient not found.")
        return

    print(f"\nPatient ID: {patient['patient_id']}")
    print(f"Name: {patient['name']}")
    print(f"Phone: {patient['phone']}")
    print(f"Age: {patient['age']}")
    if patient["prescriptions"]:
        print("\nPrescription history:")
        for i, pres in enumerate(patient["prescriptions"], 1):
            print(f"{i}. {pres['date']} - {pres['medicine']} ({pres['quantity']} units)")
    else:
        print("No prescription history yet.")

def add_prescription(data):
    patient_id = input("Enter patient ID: ").strip()
    patient = find_patient(data, patient_id)
    if not patient:
        print("Patient not found.")
        return

    medicine = input("Enter medicine name: ").strip()
    qty = input("Enter quantity: ").strip()
    if not medicine or not qty:
        print("Medicine name and quantity are required.")
        return

    for med in data["inventory"]:
        if med["name"].lower() == medicine.lower():
            if med["quantity"] < int(qty):
                print("Not enough stock. Available stock:", med["quantity"])
                return
            break
    else:
        print("Medicine not found in stock.")
        return

    prescription = {
        "date": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
        "medicine": medicine,
        "quantity": int(qty)
    }
    patient["prescriptions"].append(prescription)

    for med in data["inventory"]:
        if med["name"].lower() == medicine.lower():
            med["quantity"] -= int(qty)

    save_data(data)
    print("Prescription added successfully.")

def sell_medicine(data):
    patient_id = input("Enter patient ID: ").strip()
    patient = find_patient(data, patient_id)
    if not patient:
        print("Patient not found.")
        return

    medicine = input("Enter medicine name: ").strip()
    qty = input("Enter quantity: ").strip()

    for med in data["inventory"]:
        if med["name"].lower() == medicine.lower():
            if med["quantity"] < int(qty):
                print("Not enough stock.")
                return
            total = med["price"] * int(qty)
            med["quantity"] -= int(qty)
            save_data(data)

            print("\nReceipt")
            print("-----------")
            print(f"Patient ID: {patient['patient_id']}")
            print(f"Patient Name: {patient['name']}")
            print(f"Medicine: {med['name']}")
            print(f"Quantity: {qty}")
            print(f"Price per unit: ${med['price']:.2f}")
            print(f"Total: ${total:.2f}")
            print("-----------")
            print("Payment received.")
            return

    print("Medicine not found in stock.")

def display_menu():
    print("\n============================")
    print("PHARMACY RECEPTION SYSTEM")
    print("============================")
    print("1. Register Patient")
    print("2. Add Medicine to Stock")
    print("3. View Inventory")
    print("4. Search Patient")
    print("5. Add Prescription")
    print("6. Sell Medicine")
    print("7. Exit")

def main():
    data = load_data()

    while True:
        display_menu()
        choice = input("Choose an option: ").strip()

        if choice == "1":
            add_patient(data)
        elif choice == "2":
            add_medicine(data)
        elif choice == "3":
            view_stock(data)
        elif choice == "4":
            search_patient(data)
        elif choice == "5":
            add_prescription(data)
        elif choice == "6":
            sell_medicine(data)
        elif choice == "7":
            print("Goodbye!")
            break
        else:
            print("Invalid option. Please try again.")

if __name__ == "__main__":
    main()
