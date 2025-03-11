# Two-factorAuthentication
import tkinter as tk
from tkinter import messagebox
import pyotp
import qrcode
from PIL import Image, ImageTk
import os

# Step 1: Generate or retrieve a TOTP secret key
def get_totp_secret():
    if os.path.exists("totp_secret.txt"):
        with open("totp_secret.txt", "r") as f:
            return f.read()
    else:
        secret = pyotp.random_base32()
        with open("totp_secret.txt", "w") as f:
            f.write(secret)
        return secret

totp_secret = get_totp_secret()
totp = pyotp.TOTP(totp_secret)

# Function to generate QR Code
def generate_qr():
    uri = totp.provisioning_uri(name="YourApp@UserEmail.com", issuer_name="YourApp")
    qr = qrcode.make(uri)
    qr.save("totp_qr.png")
    return "totp_qr.png"

# Function to verify the entered code
def verify_code():
    entered_code = code_entry.get()
    if totp.verify(entered_code, valid_window=1):  # Allow a small time window tolerance
        messagebox.showinfo("Success", "Authentication successful!")
    else:
        messagebox.showerror("Error", "Invalid code. Please try again.")

# Step 2: Build the UI with tkinter
def create_ui():
    # Generate QR code and save as an image
    qr_path = generate_qr()

    # Create main UI window
    root = tk.Tk()
    root.title("Two-Factor Authentication")
    root.geometry("400x500")
    root.configure(bg="#f4f4f9")

    # Header Label
    header = tk.Label(root, text="Two-Factor Authentication", font=("Arial", 16, "bold"), bg="#f4f4f9")
    header.pack(pady=20)

    # QR Code Display
    img = Image.open(qr_path)

    qr_image = ImageTk.PhotoImage(img)
    qr_label = tk.Label(root, image=qr_image, bg="#f4f4f9")
    qr_label.image = qr_image  # Keep reference to prevent garbage collection
    qr_label.pack(pady=10)

    # Prompt Label
    prompt_label = tk.Label(root, text="Enter the 6-digit code from your authenticator app:", bg="#f4f4f9")
    prompt_label.pack(pady=10)

    # Code Entry Field
    global code_entry
    code_entry = tk.Entry(root, font=("Arial", 14), width=10, justify="center")
    code_entry.pack(pady=10)

    # Verify Button
    verify_button = tk.Button(root, text="Verify", command=verify_code, bg="#4CAF50", fg="white", font=("Arial", 12))
    verify_button.pack(pady=20)

    root.mainloop()

# Start the UI
if __name__ == "__main__":
    create_ui()
