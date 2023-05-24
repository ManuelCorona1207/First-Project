# OOPROG #Java
This project is a simple mobile wallet app where in user can top up on account , send money to other accounts and transfer to different accounts and view account info using Java Swing JOptionPane

import javax.swing.JOptionPane;
import java.util.ArrayList;

public class GCashApp {
    private static ArrayList<GCash> users = new ArrayList<>();

    public static void main(String[] args) {
        initializeUsers();
        showWelcomeForm();
    }

    private static void initializeUsers() {
        users.add(new GCash("Joaquin","12345", "2222"));
        users.add(new GCash("Patino", "54321", "0101"));
        users.add(new GCash("Juan", "00022", "2002"));
        users.add(new GCash("Dela Cruz", "00044", "2010"));
        users.add(new GCash("Janice", "00077", "8888"));
        for (GCash user : users) {
        user.setVerified(true);
    }
    }

    private static void showWelcomeForm() {
        String input = JOptionPane.showInputDialog(null, "Welcome to GCash\nEnter your number:", "GCash", JOptionPane.PLAIN_MESSAGE);
        if (input == null) {
            System.exit(0);
        } else {
            GCash user = getUserByNumber(input);
            if (user != null) {
                showPinForm(user);
            } else {
                JOptionPane.showMessageDialog(null, "GCash account not found. Please try again.", "Error", JOptionPane.ERROR_MESSAGE);
                showWelcomeForm();
            }
        }
    }

    private static void showPinForm(GCash user) {
        String input = JOptionPane.showInputDialog(null, "Account: " + user.getNumber() + "\nEnter your pin:", "GCash", JOptionPane.PLAIN_MESSAGE);
        if (input == null) {
            showWelcomeForm();
        } else if (input.equals(user.getPin())) {
            showMainForm(user);
        } else {
            JOptionPane.showMessageDialog(null, "Pin entered was invalid. Please try again.", "Error", JOptionPane.ERROR_MESSAGE);
            showPinForm(user);
        }
    }

    private static void showMainForm(GCash user) {
        String[] options = {"Top Up", "Send Money", "Display User Info", "Log Out"};
        int choice = JOptionPane.showOptionDialog(null, "Welcome, " + user.getName() + "!\nAccount: " + user.getNumber(),
                "GCash", JOptionPane.DEFAULT_OPTION, JOptionPane.PLAIN_MESSAGE, null, options, options[0]);

        switch (choice) {
            case 0:
                showTopUpForm(user);
                break;
            case 1:
                showSendMoneyForm(user);
                break;
            case 2:
                showUserInfo(user);
                break;
            case 3:
                showWelcomeForm();
                break;
            default:
                System.exit(0);
        }
    }

    private static void showTopUpForm(GCash user) {
        if (!user.isVerified()) {
            JOptionPane.showMessageDialog(null, "GCash account is not verified. Top-up is not allowed.", "Error", JOptionPane.ERROR_MESSAGE);
            showMainForm(user);
            return;
        }

        String input = JOptionPane.showInputDialog(null, "Enter amount to top up:", "GCash - Top Up", JOptionPane.PLAIN_MESSAGE);
        if (input == null) {
            showMainForm(user);
        } else {
            double amount;
            try {
                amount = Double.parseDouble(input);
                if (amount > 10000) {
                    JOptionPane.showMessageDialog(null, "Amount for top-up limit exceeded.", "Error", JOptionPane.ERROR_MESSAGE);
                    showTopUpForm(user);
                } else if (user.getBalance() + amount < 0) {
                    JOptionPane.showMessageDialog(null, "Invalid Amount", "Error", JOptionPane.ERROR_MESSAGE);
                    showTopUpForm(user);
                } else if (user.getBalance() + amount > 10000) {
                    JOptionPane.showMessageDialog(null, "Wallet balance exceeds the limit of 10,000.", "Error", JOptionPane.ERROR_MESSAGE);
                    showTopUpForm(user);
                } else {
                    user.topUp(amount);
                    JOptionPane.showMessageDialog(null, "Top up successful!", "GCash - Top Up", JOptionPane.INFORMATION_MESSAGE);
                    showMainForm(user);
                }
            } catch (NumberFormatException e) {
                JOptionPane.showMessageDialog(null, "Invalid amount. Please try again.", "Error", JOptionPane.ERROR_MESSAGE);
                showTopUpForm(user);
            }
        }
    }

   private static void showSendMoneyForm(GCash sender) {
    if (!sender.isVerified()) {
        JOptionPane.showMessageDialog(null, "GCash account is not verified. Sending money is not allowed.", "Error", JOptionPane.ERROR_MESSAGE);
        showMainForm(sender);
        return;
    }

    String input = JOptionPane.showInputDialog(null, "Enter amount to send:", "GCash - Send Money", JOptionPane.PLAIN_MESSAGE);
    if (input == null) {
        showMainForm(sender);
    } else {
        double amount;
        try {
            amount = Double.parseDouble(input);
            if (amount <= 0) {
                JOptionPane.showMessageDialog(null, "Invalid amount. Please try again.", "Error", JOptionPane.ERROR_MESSAGE);
                showSendMoneyForm(sender);  // Return to the "Enter amount" tab
            } else if (amount > 10000) {
                JOptionPane.showMessageDialog(null, "Amount to send exceeds the limit of 10,000.", "Error", JOptionPane.ERROR_MESSAGE);
                showSendMoneyForm(sender);  // Return to the "Enter amount" tab
            } else if (sender.getTotalAmountSent() + amount > 10000) {
                JOptionPane.showMessageDialog(null, "Total amount sent exceeds the limit of 10,000.", "Error", JOptionPane.ERROR_MESSAGE);
                showSendMoneyForm(sender);  // Return to the "Enter amount" tab
            } else if (sender.hasSufficientBalance(amount)) {
                showRecipientNumberForm(sender, amount);
            } else {
                JOptionPane.showMessageDialog(null, "Insufficient balance. Please enter a different amount.", "Error", JOptionPane.ERROR_MESSAGE);
                showSendMoneyForm(sender);  // Return to the "Enter amount" tab
            }
        } catch (NumberFormatException e) {
            JOptionPane.showMessageDialog(null, "Invalid amount. Please try again.", "Error", JOptionPane.ERROR_MESSAGE);
            showSendMoneyForm(sender);  // Return to the "Enter amount" tab
        }
    }
}

private static void showRecipientNumberForm(GCash sender, double amount) {
    String recipientNumber = JOptionPane.showInputDialog(null, "Enter recipient's account number:", "GCash - Send Money", JOptionPane.PLAIN_MESSAGE);
    if (recipientNumber == null) {
        showMainForm(sender);
    } else {
        GCash recipient = getUserByNumber(recipientNumber);
        if (recipient != null) {
            if (recipient.getTotalAmountReceived() + amount > 10000) {
                JOptionPane.showMessageDialog(null, "Total amount received exceeds the limit of 10,000 for the recipient.", "Error", JOptionPane.ERROR_MESSAGE);
                showRecipientNumberForm(sender, amount);  // Return to the "Enter recipient number" tab
            } else {
                sender.send(amount, recipient);
                JOptionPane.showMessageDialog(null, "Money sent successfully!", "GCash - Send Money", JOptionPane.INFORMATION_MESSAGE);
                showMainForm(sender);
            }
        } else {
            JOptionPane.showMessageDialog(null, "Recipient account not found. Please enter a valid account number.", "Error", JOptionPane.ERROR_MESSAGE);
            showRecipientNumberForm(sender, amount);  // Return to the "Enter recipient number" tab
        }
    }
}
    private static GCash getUserByNumber(String number) {
        for (GCash user : users) {
            if (user.getNumber().equals(number)) {
                return user;
            }
        }
        return null;
    }

    private static void showUserInfo(GCash user) {
        if (user.isVerified()) {
            String walletStatus = "Verified";
            String info = "Owner Name: " + user.getName() + "\nMobile No: " + user.getNumber()
                    + "\nBalance: " + user.getBalance() + "\nTotal Amount Sent: " + user.getTotalAmountSent()
                    + "\nTotal Amount Received: " + user.getTotalAmountReceived() + "\nWallet Status: " + walletStatus;

            JOptionPane.showMessageDialog(null, info, "GCash - User Info", JOptionPane.INFORMATION_MESSAGE);
            showMainForm(user);
        } else {
            JOptionPane.showMessageDialog(null, "GCash account is not verified.", "Error", JOptionPane.ERROR_MESSAGE);
            showMainForm(user);
        }
    }
}

class GCash {
    private String name;
    private String number;
    private String pin;
    private double balance;
    private double totalAmountSent;
    private double totalAmountReceived;
    private boolean verified;

    public GCash(String name, String number, String pin) {
        this.name = name;
        this.number = number;
        this.pin = pin;
        this.balance = 0.0;
        this.totalAmountSent = 0.0;
        this.totalAmountReceived = 0.0;
        this.verified = false;
    }

    public String getName() {
        return name;
    }

    public String getNumber() {
        return number;
    }

    public String getPin() {
        return pin;
    }

    public double getBalance() {
        return balance;
    }
   
    public void setTotalAmountReceived(double amount) {
        totalAmountReceived = amount;
    }

    public double getTotalAmountSent() {
        return totalAmountSent;
    }

    public double getTotalAmountReceived() {
        return totalAmountReceived;
    }

    public boolean isVerified() {
        return verified;
    }

    public void setVerified(boolean verified) {
        this.verified = verified;
    }

    public void topUp(double amount) {
        balance += amount;
    }

    public boolean hasSufficientBalance(double amount) {
        return balance >= amount;
    }

    public void send(double amount, GCash recipient) {
        balance -= amount;
        totalAmountSent += amount;
        recipient.receive(amount);
        recipient.setTotalAmountReceived(recipient.getTotalAmountReceived() + amount);
    }

    public void receive(double amount) {
        balance += amount;
    }
}
