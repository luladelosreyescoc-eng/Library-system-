from datetime import datetime

students = {}
librarian_account = {"admin": "12345"}  # default librarian account (username: admin, password: 12345)

books = {
    "1001": {"title": "Harry Potter", "available": True, "shelf": "Shelf A"},
    "1002": {"title": "Snow White", "available": False, "shelf": "Shelf B"},
    "1003": {"title": "Peter Pan", "available": True, "shelf": "Shelf A"},
    "1004": {"title": "Cinderella", "available": False, "shelf": "Shelf B"},
    "1005": {"title": "Little Red Riding Hood", "available": True, "shelf": "Shelf B"},
    "1006": {"title": "Hansel and Gretel", "available": False, "shelf": "Shelf C"},
    "1007": {"title": "The Witch Hunter", "available": True, "shelf": "Shelf C"},
    "1008": {"title": "The Chronicles of Narnia", "available": False, "shelf": "Shelf D"},
    "1009": {"title": "The God Father", "available": True, "shelf": "Shelf E"},
    "1010": {"title": "The Wizard of Oz", "available": False, "shelf": "Shelf A"}
}

borrowed_books = {}

# ---------------- STUDENT FUNCTIONS ----------------

def register_student():
    username = input("Enter username: ")
    password = input("Enter Password: ")
    if not username or not password:
        print("Please enter both username and password.\n")
        return
    if username in students:
        print("Account already exists!\n")
    else:
        students[username] = password
        print("students registered successfully!\n")

def login_student():
    username = input("Username: ")
    password = input("Enter Password: ")

    if username in students and students[username] == password:
        print("✅ Login successful!\n")
        student_menu(username)
    else:
        print("❌ Invalid credentials. Try again or register.\n")

def show_books():
    print("\n📚 Available Books:")
    print("------------------------------------------------------------")
    print("{:<8} {:<30} {:<12} {:<10}".format("ID", "Title", "Status", "Shelf"))
    print("------------------------------------------------------------")
    for book_id, info in sorted(books.items()):
        status = "Available" if info.get("available", False) else "Unavailable"
        shelf = info.get("shelf", "Unknown")
        print("{:<8} {:<30} {:<12} {:<10}".format(book_id, info.get("title", "Unknown Title"), status, shelf))
    print("------------------------------------------------------------\n")

def borrow_book(username):
    show_books()
    book_id = input("Enter Book ID to borrow (or type 'cancel' to go back): ")

    if not book_id:
        print("Please enter a Book ID.\n")
        return

    if book_id.lower() == "cancel":
        print("❌ Borrowing canceled.\n")
        return

    if book_id in books:
        if books[book_id]["available"]:
            confirm = input(f"Are you sure you want to borrow '{books[book_id]['title']}'? (yes/no): ").lower()
            if confirm != "yes":
                print("❌ Borrowing canceled.\n")
                return

            books[book_id]["available"] = False
            borrow_date = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
            borrowed_books.setdefault(username, []).append({
                "book_id": book_id,
                "borrow_date": borrow_date,
                "return_date": None
            })
            print(f"✅ You borrowed '{books[book_id]['title']}' on {borrow_date}\n")
        else:
            print("❌ Sorry, this book is already borrowed (Unavailable).\n")
    else:
        print("❌ Book not found.\n")

def return_book(username):
    user_borrowed = borrowed_books.get(username, [])
    if not user_borrowed:
        print("📭 You have no borrowed books.\n")
        return

    print("📘 Your borrowed books:")
    for record in user_borrowed:
        print(f"- {record['book_id']}: {books.get(record['book_id'], {}).get('title', 'Unknown Title')} (Borrowed: {record['borrow_date']})")
    print()

    book_id = input("Enter Book ID to return: ")
    if not book_id:
        print("Please enter a Book ID.\n")
        return

    for record in user_borrowed:
        if record["book_id"] == book_id and record["return_date"] is None:
            record["return_date"] = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
            books[book_id]["available"] = True
            print(f"✅ You returned '{books[book_id]['title']}' on {record['return_date']}\n")
            return

    print("❌ Invalid Book ID (not in your borrowed list or already returned).\n")

def view_borrow_history(username):
    records = borrowed_books.get(username, [])
    if not records:
        print("📭 You haven't borrowed any books yet.\n")
        return

    print("\n📖 Borrow History:")
    print("----------------------------")
    for r in records:
        title = books.get(r["book_id"], {}).get("title", "Unknown Title")
        status = "✅ Returned" if r["return_date"] else "⏳ Not Returned"
        print(f"{r['book_id']} - {title}")
        print(f"   Borrowed: {r['borrow_date']}")
        print(f"   Returned: {r['return_date'] or '---'} ({status})")
        print("----------------------------")
    print()

def student_menu(username):
    while True:
        print("=== Student Menu ===")
        print("1. View All Books")
        print("2. Borrow Book")
        print("3. Return Book")
        print("4. View Borrow History")
        print("5. Log Out")
        choice = input("Select option: ")
        if choice == "1":
            show_books()
        elif choice == "2":
            borrow_book(username)
        elif choice == "3":
            return_book(username)
        elif choice == "4":
            view_borrow_history(username)
        elif choice == "5":
            print("👋 Logged out.\n")
            break
        else:
            print("❌ Invalid choice.\n")

# ---------------- LIBRARIAN FUNCTIONS ----------------

def login_librarian():
    username = input("Librarian username: ")
    password = input("Enter Password: ")

    if username in librarian_account and librarian_account[username] == password:
        print("✅ Librarian login successful!\n")
        librarian_menu()
    else:
        print("❌ Invalid librarian credentials.\n")

def librarian_menu():
    while True:
        print("=== Librarian Menu ===")
        print("1. View All Books")
        print("2. Add New Book")
        print("3. View All Borrowed Books")
        print("4. Log Out")
        choice = input("Select option: ")

        if choice == "1":
            show_books()
        elif choice == "2":
            add_book()
        elif choice == "3":
            view_all_borrowed_books()
        elif choice == "4":
            print("👋 Logged out of librarian account.\n")
            break
        else:
            print("❌ Invalid choice.\n")

def add_book():
    new_id = input("Enter new Book ID: ")
    title = input("Enter Book Title: ")
    shelf = input("Enter Shelf Location (e.g., Shelf A): ")

    if not new_id or not title or not shelf:
        print("⚠️ Book ID, title, and shelf cannot be empty.\n")
        return

    if new_id in books:
        print("⚠️ Book ID already exists.\n")
        return

    books[new_id] = {"title": title, "available": True, "shelf": shelf}
    print(f"✅ Book '{title}' added successfully to {shelf} with ID {new_id}!\n")


def view_all_borrowed_books():
    print("\n📚 All Borrowed Books:")
    print("----------------------------")
    if not borrowed_books:
        print("No books have been borrowed yet.\n")
        return

    for user, records in borrowed_books.items():
        print(f"👤 {user}:")
        for r in records:
            title = books.get(r["book_id"], {}).get("title", "Unknown Title")
            status = "✅ Returned" if r["return_date"] else "⏳ Not Returned"
            print(f"   {r['book_id']} - {title}")
            print(f"   Borrowed: {r['borrow_date']}")
            print(f"   Returned: {r['return_date'] or '---'} ({status})")
        print("----------------------------")
    print()

# ---------------- MAIN MENU ----------------

def main():
    while True:
        print("=== Library System ===")
        print("1. Register Student")
        print("2. Student Login")
        print("3. Librarian Login")
        print("4. Exit")
        choice = input("Choose option: ")

        if choice == "1":
            register_student()
        elif choice == "2":
            login_student()
        elif choice == "3":
            login_librarian()
        elif choice == "4":
            print("📕 Goodbye!")
            break
        else:
            print("❌ Invalid choice.\n")

if __name__ == "__main__":
    main()