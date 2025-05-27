import java.util.Scanner;
import java.util.ArrayList;
import java.util.List;

class InsufficientBalanceException extends Exception {
    public InsufficientBalanceException(String message) {
        super(message);
    }

    public InsufficientBalanceException() {
        super("Insufficient balance in your account.");
    }
}

abstract class BankAccount {
    private String accountHolderName;
    private String accountNumber;
    protected double balance;

    public BankAccount(String accountHolderName, String accountNumber, double balance) {
        this.accountHolderName = accountHolderName;
        this.accountNumber = accountNumber;
        this.balance = balance;
    }

    public String getAccountHolderName() {
        return accountHolderName;
    }

    public String getAccountNumber() {
        return accountNumber;
    }

    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
            System.out.println("Deposit successful.");
        } else {
            System.out.println("Deposit amount must be positive.");
        }
    }

    public abstract void withdraw(double amount) throws InsufficientBalanceException;

    public abstract void displayAccountInfo();
}

class SavingsAccount extends BankAccount {
    private double interestRate = 0.05;

    public SavingsAccount(String accountHolderName, String accountNumber, double balance) {
        super(accountHolderName, accountNumber, balance);
    }

    public void addInterest() {
        double interest = balance * interestRate;
        balance += interest;
        System.out.println("Interest added: " + interest);
    }

    @Override
    public void withdraw(double amount) throws InsufficientBalanceException {
        if (amount > balance) {
            throw new InsufficientBalanceException("Withdrawal failed: Insufficient balance.");
        }
        balance -= amount;
        System.out.println("Withdrawal successful.");
    }

    @Override
    public void displayAccountInfo() {
        System.out.println("Savings Account - " + getAccountNumber());
        System.out.println("Holder: " + getAccountHolderName());
        System.out.println("Balance: Rs. " + getBalance());
    }
}

class CurrentAccount extends BankAccount {
    private double overdraftLimit = 5000;

    public CurrentAccount(String accountHolderName, String accountNumber, double balance) {
        super(accountHolderName, accountNumber, balance);
    }

    @Override
    public void withdraw(double amount) throws InsufficientBalanceException {
        if (amount > (balance + overdraftLimit)) {
            throw new InsufficientBalanceException("Withdrawal failed: Overdraft limit exceeded.");
        }
        balance -= amount;
        System.out.println("Withdrawal successful.");
    }

    @Override
    public void displayAccountInfo() {
        System.out.println("Current Account - " + getAccountNumber());
        System.out.println("Holder: " + getAccountHolderName());
        System.out.println("Balance: Rs. " + getBalance());
        System.out.println("Overdraft Limit: Rs. " + overdraftLimit);
    }
}

class Customer {
    private String name;
    private List<BankAccount> accounts;

    public Customer(String name) {
        this.name = name;
        this.accounts = new ArrayList<>();
    }

    public String getName() {
        return name;
    }

    public void addAccount(BankAccount account) {
        accounts.add(account);
    }

    public List<BankAccount> getAccounts() {
        return accounts;
    }

    public BankAccount getAccountByIndex(int index) {
        if (index >= 0 && index < accounts.size()) {
            return accounts.get(index);
        }
        return null;
    }

    public void displayAllAccounts() {
        for (int i = 0; i < accounts.size(); i++) {
            System.out.println("\nAccount " + (i + 1) + ":");
            accounts.get(i).displayAccountInfo();
        }
    }
}

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.println("Welcome to Rural Bank of Nepal");
        System.out.print("Enter customer name: ");
        String name = scanner.nextLine();

        Customer customer = new Customer(name);

        SavingsAccount savings = new SavingsAccount(name, "SAV001", 0);
        CurrentAccount current = new CurrentAccount(name, "CUR001", 0);

        customer.addAccount(savings);
        customer.addAccount(current);

        int choice;
        do {
            System.out.println("\nChoose operation:");
            System.out.println("1. Deposit");
            System.out.println("2. Withdraw");
            System.out.println("3. Add Interest (Savings Only)");
            System.out.println("4. View Accounts");
            System.out.println("5. Exit");
            System.out.print("Enter choice: ");
            while (!scanner.hasNextInt()) {
                System.out.print("Invalid input. Enter a number: ");
                scanner.next();
            }
            choice = scanner.nextInt();

            switch (choice) {
                case 1 -> {
                    System.out.print("Select account (1: Savings, 2: Current): ");
                    int accType = scanner.nextInt();
                    System.out.print("Enter deposit amount: ");
                    double amount = scanner.nextDouble();
                    BankAccount acc = customer.getAccountByIndex(accType - 1);
                    if (acc != null) {
                        acc.deposit(amount);
                    } else {
                        System.out.println("Invalid account selected.");
                    }
                }
                case 2 -> {
                    System.out.print("Select account (1: Savings, 2: Current): ");
                    int accType = scanner.nextInt();
                    System.out.print("Enter withdrawal amount: ");
                    double amount = scanner.nextDouble();
                    BankAccount acc = customer.getAccountByIndex(accType - 1);
                    if (acc != null) {
                        try {
                            acc.withdraw(amount);
                        } catch (InsufficientBalanceException e) {
                            System.out.println(e.getMessage());
                        }
                    } else {
                        System.out.println("Invalid account selected.");
                    }
                }
                case 3 -> savings.addInterest();
                case 4 -> customer.displayAllAccounts();
                case 5 -> System.out.println("Thank you for using Rural Bank of Nepal.");
                default -> System.out.println("Invalid choice. Please try again.");
            }
        } while (choice != 5);

        scanner.close();
    }
}
