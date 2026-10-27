import java.time.LocalDate;
import java.time.temporal.ChronoUnit;
import java.util.*;

class Author {
    private final String name;
    private final String biography;

    public Author(String name, String biography) {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Author name is required.");
        }
        this.name = name.trim();
        this.biography = biography == null ? "" : biography.trim();
    }

    public String getName() { return name; }
    public String getBiography() { return biography; }

    @Override
    public String toString() { return name; }
}

class Book {
    private final int id;
    private final String title;
    private final Author author;
    private final int totalCopies;
    private int availableCopies;

    public Book(int id, String title, Author author, int copies) {
        if (title == null || title.isBlank()) {
            throw new IllegalArgumentException("Book title is required.");
        }
        if (author == null) {
            throw new IllegalArgumentException("Author is required.");
        }
        if (copies < 1) {
            throw new IllegalArgumentException("Copies must be at least 1.");
        }

        this.id = id;
        this.title = title.trim();
        this.author = author;
        this.totalCopies = copies;
        this.availableCopies = copies;
    }

    public int getId() { return id; }
    public String getTitle() { return title; }
    public Author getAuthor() { return author; }
    public int getTotalCopies() { return totalCopies; }
    public int getAvailableCopies() { return availableCopies; }
    public boolean isAvailable() { return availableCopies > 0; }

    public void borrowCopy() {
        if (!isAvailable()) {
            throw new IllegalStateException("No available copies.");
        }
        availableCopies--;
    }

    public void returnCopy() {
        if (availableCopies >= totalCopies) {
            throw new IllegalStateException("All copies are already in stock.");
        }
        availableCopies++;
    }

    @Override
    public String toString() {
        return String.format("B%d | %s | Author: %s | Available: %d/%d",
                id, title, author.getName(), availableCopies, totalCopies);
    }
}

class Member {
    private final int id;
    private final String name;
    private final int maxBooks;
    private final int loanDays;
    private final List<BorrowTransaction> activeLoans = new ArrayList<>();

    public Member(int id, String name) {
        this(id, name, 3, 14);
    }

    protected Member(int id, String name, int maxBooks, int loanDays) {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Member name is required.");
        }
        this.id = id;
        this.name = name.trim();
        this.maxBooks = maxBooks;
        this.loanDays = loanDays;
    }

    public int getId() { return id; }
    public String getName() { return name; }
    public int getMaxBooks() { return maxBooks; }
    public int getLoanDays() { return loanDays; }

    public List<BorrowTransaction> getActiveLoans() {
        return Collections.unmodifiableList(activeLoans);
    }

    void addLoan(BorrowTransaction transaction) {
        activeLoans.add(transaction);
    }

    void closeLoan(BorrowTransaction transaction) {
        activeLoans.remove(transaction);
    }

    public boolean canBorrow() {
        return activeLoans.size() < maxBooks;
    }

    @Override
    public String toString() {
        return String.format("M%d | %s | Active loans: %d/%d | Loan period: %d days",
                id, name, activeLoans.size(), maxBooks, loanDays);
    }
}

class PremiumMember extends Member {
    public PremiumMember(int id, String name) {
        super(id, name, 5, 30);
    }

    @Override
    public String toString() {
        return "PREMIUM " + super.toString();
    }
}

class BorrowTransaction {
    private final int id;
    private final Book book;
    private final Member member;
    private final LocalDate borrowDate;
    private final LocalDate dueDate;
    private boolean returned;

    public BorrowTransaction(int id, Book book, Member member, LocalDate borrowDate) {
        this.id = id;
        this.book = book;
        this.member = member;
        this.borrowDate = borrowDate;
        this.dueDate = borrowDate.plusDays(member.getLoanDays());
        this.returned = false;
    }

    public int getId() { return id; }
    public Book getBook() { return book; }
    public Member getMember() { return member; }
    public LocalDate getBorrowDate() { return borrowDate; }
    public LocalDate getDueDate() { return dueDate; }
    public boolean isReturned() { return returned; }
    public void markReturned() { returned = true; }

    @Override
    public String toString() {
        return String.format("BR%d | %s | Member: %s | Borrowed: %s | Due: %s",
                id, book.getTitle(), member.getName(), borrowDate, dueDate);
    }
}

class ReturnTransaction {
    private final int id;
    private final BorrowTransaction borrowTransaction;
    private final LocalDate returnDate;
    private final long lateDays;
    private final double fine;

    public ReturnTransaction(int id, BorrowTransaction borrowTransaction,
                             LocalDate returnDate, double finePerDay) {
        this.id = id;
        this.borrowTransaction = borrowTransaction;
        this.returnDate = returnDate;
        this.lateDays = Math.max(0,
                ChronoUnit.DAYS.between(borrowTransaction.getDueDate(), returnDate));
        this.fine = lateDays * finePerDay;
    }

    public int getId() { return id; }
    public BorrowTransaction getBorrowTransaction() { return borrowTransaction; }
    public LocalDate getReturnDate() { return returnDate; }
    public long getLateDays() { return lateDays; }
    public double getFine() { return fine; }

    public void printReceipt() {
        System.out.println("\n========== BOOK RETURN RECEIPT ==========");
        System.out.println("Return ID   : RT" + id);
        System.out.println("Book        : " + borrowTransaction.getBook().getTitle());
        System.out.println("Member      : " + borrowTransaction.getMember().getName());
        System.out.println("Due date    : " + borrowTransaction.getDueDate());
        System.out.println("Return date : " + returnDate);
        System.out.println("Late days   : " + lateDays);
        System.out.printf("Fine        : Rs. %.2f%n", fine);
        System.out.println("=========================================");
    }
}

class Library {
    private final Map<Integer, Book> books = new LinkedHashMap<>();
    private final Map<Integer, Member> members = new LinkedHashMap<>();
    private final Map<Integer, BorrowTransaction> borrowTransactions = new LinkedHashMap<>();
    private final Map<Integer, ReturnTransaction> returnTransactions = new LinkedHashMap<>();
    private int nextBookId = 1;
    private int nextMemberId = 1;
    private int nextBorrowId = 1;
    private int nextReturnId = 1;
    private final double finePerDay;

    public Library(double finePerDay) {
        if (finePerDay < 0) {
            throw new IllegalArgumentException("Fine cannot be negative.");
        }
        this.finePerDay = finePerDay;
    }

    public Collection<Book> getBooks() { return books.values(); }
    public Collection<Member> getMembers() { return members.values(); }
    public Book findBook(int id) { return books.get(id); }
    public Member findMember(int id) { return members.get(id); }

    public Book addBook(String title, String authorName, String biography, int copies) {
        Author author = new Author(authorName, biography);
        Book book = new Book(nextBookId++, title, author, copies);
        books.put(book.getId(), book);
        return book;
    }

    public Member addMember(String name, boolean premium) {
        Member member;
        if (premium) {
            member = new PremiumMember(nextMemberId++, name);
        } else {
            member = new Member(nextMemberId++, name);
        }
        members.put(member.getId(), member);
        return member;
    }

    public BorrowTransaction borrowBook(int bookId, int memberId, LocalDate date) {
        Book book = books.get(bookId);
        Member member = members.get(memberId);

        if (book == null) {
            throw new IllegalArgumentException("Book ID not found.");
        }
        if (member == null) {
            throw new IllegalArgumentException("Member ID not found.");
        }
        if (!book.isAvailable()) {
            throw new IllegalStateException("Book is out of stock.");
        }
        if (!member.canBorrow()) {
            throw new IllegalStateException("Member has reached the borrowing limit.");
        }

        book.borrowCopy();
        BorrowTransaction transaction =
                new BorrowTransaction(nextBorrowId++, book, member, date);
        borrowTransactions.put(transaction.getId(), transaction);
        member.addLoan(transaction);
        return transaction;
    }

    public ReturnTransaction returnBook(int borrowId, LocalDate date) {
        BorrowTransaction borrow = borrowTransactions.get(borrowId);

        if (borrow == null) {
            throw new IllegalArgumentException("Borrow transaction ID not found.");
        }
        if (borrow.isReturned()) {
            throw new IllegalStateException("This book has already been returned.");
        }

        ReturnTransaction transaction =
                new ReturnTransaction(nextReturnId++, borrow, date, finePerDay);
        borrow.markReturned();
        borrow.getBook().returnCopy();
        borrow.getMember().closeLoan(borrow);
        returnTransactions.put(transaction.getId(), transaction);
        return transaction;
    }

    public void displayAvailableBooks() {
        System.out.println("\n---------- AVAILABLE BOOKS ----------");
        boolean found = false;
        for (Book book : books.values()) {
            if (book.isAvailable()) {
                System.out.println(book);
                found = true;
            }
        }
        if (!found) {
            System.out.println("No books currently available.");
        }
    }

    public void displayMembers() {
        System.out.println("\n--------------- MEMBERS -------------");
        if (members.isEmpty()) {
            System.out.println("No members registered.");
        }
        for (Member member : members.values()) {
            System.out.println(member);
        }
    }

    public void displayActiveLoans() {
        System.out.println("\n------------- ACTIVE LOANS -----------");
        boolean found = false;
        for (BorrowTransaction transaction : borrowTransactions.values()) {
            if (!transaction.isReturned()) {
                System.out.println(transaction);
                found = true;
            }
        }
        if (!found) {
            System.out.println("No active loans.");
        }
    }
}

public class Main {
    private static final Scanner sc = new Scanner(System.in);
    private static final Library library = new Library(5.0);

    public static void main(String[] args) {
        // Sample data for demonstration. Remove these lines for an empty library.
        library.addBook("Ponniyin Selvan", "Kalki Krishnamurthy",
                "Tamil historical novel", 3);
        library.addBook("Wings of Fire", "A. P. J. Abdul Kalam",
                "Autobiography", 2);
        library.addMember("Arun", false);
        library.addMember("Priya", true);

        menu();
    }

    private static void menu() {
        while (true) {
            System.out.println("\n========== LIBRARY MANAGEMENT SYSTEM ==========");
            System.out.println("1. Add Book");
            System.out.println("2. Add Member");
            System.out.println("3. Borrow Book");
            System.out.println("4. Return Book");
            System.out.println("5. Display Available Books");
            System.out.println("6. Display Members");
            System.out.println("7. Display Active Loans");
            System.out.println("8. Exit");

            int choice = readInt("Enter choice: ");
            try {
                switch (choice) {
                    case 1 -> addBook();
                    case 2 -> addMember();
                    case 3 -> borrowBook();
                    case 4 -> returnBook();
                    case 5 -> library.displayAvailableBooks();
                    case 6 -> library.displayMembers();
                    case 7 -> library.displayActiveLoans();
                    case 8 -> {
                        System.out.println("Thank you for using Library Management System.");
                        return;
                    }
                    default -> System.out.println("Invalid choice. Select 1-8.");
                }
            } catch (RuntimeException ex) {
                System.out.println("ERROR: " + ex.getMessage());
            }
        }
    }

    private static void addBook() {
        System.out.print("Book title: ");
        String title = sc.nextLine().trim();
        System.out.print("Author name: ");
        String author = sc.nextLine().trim();
        System.out.print("Author biography (optional): ");
        String biography = sc.nextLine().trim();
        int copies = readInt("Number of copies: ");

        System.out.println("Book added: "
                + library.addBook(title, author, biography, copies));
    }

    private static void addMember() {
        System.out.print("Member name: ");
        String name = sc.nextLine().trim();
        System.out.print("Premium member? (y/n): ");
        boolean premium = sc.nextLine().trim().equalsIgnoreCase("y");

        System.out.println("Member added: " + library.addMember(name, premium));
    }

    private static void borrowBook() {
        library.displayAvailableBooks();
        int bookId = readInt("Book ID: ");
        library.displayMembers();
        int memberId = readInt("Member ID: ");

        BorrowTransaction transaction =
                library.borrowBook(bookId, memberId, LocalDate.now());

        System.out.println("\n========== BOOK BORROW RECEIPT ==========");
        System.out.println("Borrow ID   : BR" + transaction.getId());
        System.out.println("Book        : " + transaction.getBook().getTitle());
        System.out.println("Member      : " + transaction.getMember().getName());
        System.out.println("Borrow date : " + transaction.getBorrowDate());
        System.out.println("Due date    : " + transaction.getDueDate());
        System.out.println("Stock left  : "
                + transaction.getBook().getAvailableCopies());
        System.out.println("=========================================");
    }

    private static void returnBook() {
        library.displayActiveLoans();
        int borrowId = readInt("Borrow transaction ID (number only): ");

        ReturnTransaction transaction =
                library.returnBook(borrowId, LocalDate.now());
        transaction.printReceipt();
        System.out.println("Updated book stock: "
                + transaction.getBorrowTransaction().getBook().getAvailableCopies());
    }

    private static int readInt(String prompt) {
        while (true) {
            System.out.print(prompt);
            try {
                return Integer.parseInt(sc.nextLine().trim());
            } catch (NumberFormatException ex) {
                System.out.println("Enter a valid whole number.");
            }
        }
    }
}
