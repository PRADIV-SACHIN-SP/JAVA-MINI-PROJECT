import java.time.LocalDate;
import java.time.temporal.ChronoUnit;
import java.util.*;
/* Required classes: Author, Book, Member, PremiumMember,
   BorrowTransaction, ReturnTransaction, Library */
class Author {
    private final String name;
    private final String biography;
    public Author(String name, String biography) {
        if (name == null || name.isBlank()) throw new IllegalArgumentException("Author name is required.");
        this.name = name.trim();
        this.biography = biography == null ? "" : biography.trim();
    }
    public String getName() { return name; }
    public String getBiography() { return biography; }
    @Override public String toString() { return name; }
}
class Book {
    private final int id;
    private final String title;
    private final Author author;
    private int totalCopies;
    private int availableCopies;
    public Book(int id, String title, Author author, int copies) {
        if (title == null || title.isBlank()) throw new IllegalArgumentException("Book title is required.");
        if (author == null) throw new IllegalArgumentException("Author is required.");
        if (copies < 1) throw new IllegalArgumentException("Copies must be at least 1.");
        this.id = id; this.title = title.trim(); this.author = author;
        this.totalCopies = copies; this.availableCopies = copies;
    }
    public int getId() { return id; }
    public String getTitle() { return title; }
    public Author getAuthor() { return author; }
    public int getTotalCopies() { return totalCopies; }
    public int getAvailableCopies() { return availableCopies; }
    public boolean isAvailable() { return availableCopies > 0; }
    public void borrowCopy() {
        if (!isAvailable()) throw new IllegalStateException("No available copies.");
        availableCopies--;
    }
    public void returnCopy() {
        if (availableCopies >= totalCopies) throw new IllegalStateException("All copies are already in stock.
");
}
        availableCopies++;
    }
    @Override public String toString() {
        return String.format("B%d | %s | Author: %s | Available: %d/%d",
                id, title, author.getName(), availableCopies, totalCopies);
    }
class Member {
    private final int id;
    private final String name;
    private final int maxBooks;
    private final int loanDays;
    private final List<BorrowTransaction> activeLoans = new ArrayList<>();
    public Member(int id, String name) { this(id, name, 3, 14); }
    protected Member(int id, String name, int maxBooks, int loanDays) {
        if (name == null || name.isBlank()) throw new IllegalArgumentException("Member name is required.");
        this.id = id; this.name = name.trim(); this.maxBooks = maxBooks; this.loanDays = loanDays;
    }
    public int getId() { return id; }
    public String getName() { return name; }
    public int getMaxBooks() { return maxBooks; }
