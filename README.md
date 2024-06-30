# Bank-Application

# Project-SCD

// class DeficientFundsException
package com.bankapp;

class DeficientFundsException extends Exception {
    /**
	 * 
	 */
	private static final long serialVersionUID = 1L;

	public DeficientFundsException() {
        super("Insufficient funds for this withdrawal.");
    }

    public DeficientFundsException(String message) {
        super(message);
    }
}



// class Cl_Accounts
package com.bankapp;

class Cl_Accounts {
    private long Accnt_No;
    private String client_fname;
    private String client_lname;
    private float client_balance;
    private static long Nxt_Accnt_No = 0;

    public Cl_Accounts() {}

    public Cl_Accounts(String fname, String lname, float client_balance) {
        Nxt_Accnt_No++;
        this.Accnt_No = Nxt_Accnt_No;
        this.client_fname = fname;
        this.client_lname = lname;
        this.client_balance = client_balance;
    }

    public long getAccNo() {
        return Accnt_No;
    }

    public String getFName() {
        return client_fname;
    }

    public String getLName() {
        return client_lname;
    }

    public float getBalance() {
        return client_balance;
    }

    public void deposit(float amount) {
        client_balance += amount;
    }

    public void withdraw(float amount) throws DeficientFundsException {
        if (client_balance - amount < 100) {
            throw new DeficientFundsException();
        }
        client_balance -= amount;
    }

    public static void setLastAccntNo(long Accnt_No) {
        Nxt_Accnt_No = Accnt_No;
    }

    public static long getLastAccntNo() {
        return Nxt_Accnt_No;
    }

    @Override
    public String toString() {
        return "First Name: " + client_fname + "\n" +
               "Last Name: " + client_lname + "\n" +
               "Account Number: " + Accnt_No + "\n" +
               "Balance: " + client_balance + "\n";
    }
}


//class bank
package com.bankapp;

import java.io.*;
import java.util.*;

class Bank {
    private Map<Long, Cl_Accounts> accounts_cl;

    public Bank() {
        accounts_cl = new HashMap<>();
        loadAccounts();
    }

    public Cl_Accounts openAccount(String fname, String lname, float balance) {
        Cl_Accounts account = new Cl_Accounts(fname, lname, balance);
        accounts_cl.put(account.getAccNo(), account);
        saveAccounts();
        return account;
    }

    public Cl_Accounts balanceEnquiry(long accountNo) {
        return accounts_cl.get(accountNo);
    }

    public Cl_Accounts deposit(long accountNo, float amount) {
        Cl_Accounts account = accounts_cl.get(accountNo);
        if (account != null) {
            account.deposit(amount);
            saveAccounts();
        }
        return account;
    }

    public Cl_Accounts withdraw(long accountNo, float amount) throws DeficientFundsException {
        Cl_Accounts account = accounts_cl.get(accountNo);
        if (account != null) {
            account.withdraw(amount);
            saveAccounts();
        }
        return account;
    }

    public void closeAccount(long accountNo) {
        accounts_cl.remove(accountNo);
        saveAccounts();
    }

    public void showAllAccounts() {
        for (Cl_Accounts account : accounts_cl.values()) {
            System.out.println(account);
        }
    }

    private void saveAccounts() {
        try (PrintWriter writer = new PrintWriter(new FileWriter("Bank.data"))) {
            for (Cl_Accounts account : accounts_cl.values()) {
                writer.println(account.getAccNo());
                writer.println(account.getFName());
                writer.println(account.getLName());
                writer.println(account.getBalance());
            }
        } catch (IOException e) {
            System.out.println("Error saving accounts: " + e.getMessage());
        }
    }

    private void loadAccounts() {
        try (Scanner scanner = new Scanner(new File("Bank.data"))) {
            while (scanner.hasNextLong()) {
                long accNo = scanner.nextLong();
                String fname = scanner.next();
                String lname = scanner.next();
                float balance = scanner.nextFloat();
                Cl_Accounts account = new Cl_Accounts(fname, lname, balance);
                Cl_Accounts.setLastAccntNo(accNo - 1); // Reset the last account number to be the highest existing account number
                accounts_cl.put(accNo, account);
            }
            if (!accounts_cl.isEmpty()) {
                Cl_Accounts.setLastAccntNo(Collections.max(accounts_cl.keySet()));
            }
        } catch (FileNotFoundException e) {
            // File not found, this is fine as it might be the first run
        }
    }

    public static void main(String[] args) {
    	
        Bank bank = new Bank();
        Scanner scanner = new Scanner(System.in);
        int option;

        do {
            System.out.println("\n\tSelect One Option Below ");
            System.out.println("\n\t1 Open an Account");
            System.out.println("\n\t2 Balance Enquiry");
            System.out.println("\n\t3 Deposit");
            System.out.println("\n\t4 Withdrawal");
            System.out.println("\n\t5 Close an Account");
            System.out.println("\n\t6 Show All Accounts");
            System.out.println("\n\t7 Quit");
            System.out.print("Enter your choice: ");
            option = scanner.nextInt();

            switch (option) {
                case 1:
                    System.out.print("Enter First Name: ");
                    String fname = scanner.next();
                    System.out.print("Enter Last Name: ");
                    String lname = scanner.next();
                    System.out.print("Enter Initial Balance: ");
                    float balance = scanner.nextFloat();
                    Cl_Accounts account = bank.openAccount(fname, lname, balance);
                    System.out.println("\nCongratulations Account is Created");
                    System.out.println(account);
                    break;
                case 2:
                    System.out.print("Enter Account Number: ");
                    long accountNo = scanner.nextLong();
                    Cl_Accounts accountEnq = bank.balanceEnquiry(accountNo);
                    if (accountEnq != null) {
                        System.out.println("\nYour Account Details");
                        System.out.println(accountEnq);
                    } else {
                        System.out.println("Account not found.");
                    }
                    break;
                case 3:
                    System.out.print("Enter Account Number: ");
                    accountNo = scanner.nextLong();
                    System.out.print("Enter Amount: ");
                    float amount = scanner.nextFloat();
                    Cl_Accounts accountDep = bank.deposit(accountNo, amount);
                    if (accountDep != null) {
                        System.out.println("\nAmount is Deposited");
                        System.out.println(accountDep);
                    } else {
                        System.out.println("Account not found.");
                    }
                    break;
                case 4:
                    System.out.print("Enter Account Number: ");
                    accountNo = scanner.nextLong();
                    System.out.print("Enter Amount: ");
                    amount = scanner.nextFloat();
                    try {
                        Cl_Accounts accountWith = bank.withdraw(accountNo, amount);
                        System.out.println("\nAmount Withdrawn");
                        System.out.println(accountWith);
                    } catch (DeficientFundsException e) {
                        System.out.println("Insufficient funds for this withdrawal.");
                    } catch (NullPointerException e) {
                        System.out.println("Account not found.");
                    }
                    break;
                case 5:
                    System.out.print("Enter Account Number: ");
                    accountNo = scanner.nextLong();
                    bank.closeAccount(accountNo);
                    System.out.println("\nAccount is Closed");
                    break;
                case 6:
                    bank.showAllAccounts();
                    break;
                case 7:
                    break;
                default:
                    System.out.println("\nEnter correct choice");
                    break;
            }
        } while (option != 7);

        scanner.close();
    }
}
