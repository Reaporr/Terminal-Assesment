import javax.swing.*;
import javax.swing.table.DefaultTableModel;
import javax.swing.table.TableRowSorter;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.io.*;
import java.util.*;
import javax.swing.RowFilter;

class EmployeeApplication extends JFrame implements ActionListener {
    private final JTextField employeeNumberField;
    private final JPasswordField passwordField;
    private final JButton loginButton;
    private final JLabel employeeNumberLabel;
    private final JLabel passwordLabel;
    private final ArrayList<String[]> users;
    private final ArrayList<String[]> attendanceRecords;
    private final JPanel mainPanel;
    private final JLabel welcomeLabel;
    private final JButton attendanceButton;
    private JButton payslipButton, createEntryButton, deleteEntryButton, listEmployeesButton, searchEmployeeButton, clockInButton, clockOutButton, viewAttendanceHistoryButton, logoutButton;

    private String currentUserRole;
    private String currentEmployeeNumber;

    public EmployeeApplication() {
        // Initialize JFrame
        setTitle("Employee Login");
        setSize(300, 150);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);
        setLayout(new GridLayout(3, 2));

        // Initialize components
        employeeNumberLabel = new JLabel("Employee Number:");
        add(employeeNumberLabel);
        employeeNumberField = new JTextField();
        add(employeeNumberField);

        passwordLabel = new JLabel("Password:");
        add(passwordLabel);
        passwordField = new JPasswordField();
        add(passwordField);

        loginButton = new JButton("Login");
        loginButton.addActionListener(this);
        add(loginButton);

        // Load users and attendance from CSV
        users = loadUsersFromCSV("users.csv");
        attendanceRecords = loadAttendanceFromCSV("attendance.csv");

        // Initialize main panel
        mainPanel = new JPanel();
        mainPanel.setLayout(new BorderLayout());
        add(mainPanel);

        // Initialize components for employee information
        welcomeLabel = new JLabel();
        mainPanel.add(welcomeLabel, BorderLayout.CENTER);

        attendanceButton = new JButton("Attendance");
        attendanceButton.addActionListener(this);

        payslipButton = new JButton("Payslip");
        payslipButton.addActionListener(this);

        createEntryButton = new JButton("Create Entry");
        createEntryButton.addActionListener(this);

        deleteEntryButton = new JButton("Delete Entry");
        deleteEntryButton.addActionListener(this);

        listEmployeesButton = new JButton("List Employees");
        listEmployeesButton.addActionListener(this);

        searchEmployeeButton = new JButton("Search Employee");
        searchEmployeeButton.addActionListener(this);

        clockInButton = new JButton("Clock In");
        clockInButton.addActionListener(this);

        clockOutButton = new JButton("Clock Out");
        clockOutButton.addActionListener(this);

        viewAttendanceHistoryButton = new JButton("View Attendance History");
        viewAttendanceHistoryButton.addActionListener(this);

        logoutButton = new JButton("Logout");
        logoutButton.addActionListener(this);
    }

    // Action performed when a button is clicked
    public void actionPerformed(ActionEvent e) {
        if (e.getSource() == loginButton) {
            String employeeNumber = employeeNumberField.getText();
            String password = new String(passwordField.getPassword());

            if (employeeNumber.isEmpty() || password.isEmpty()) {
                JOptionPane.showMessageDialog(this, "Please enter both employee number and password.", "Error", JOptionPane.ERROR_MESSAGE);
                return;
            }

            boolean found = false;
            for (String[] user : users) {
                if (user[0].equals(employeeNumber) && user[19].equals(password)) {
                    found = true;
                    currentEmployeeNumber = employeeNumber;
                    currentUserRole = user[20]; // Assuming user[3] contains the role: 'admin' or 'user'
                    displayMainPanel(user[1], user[0]);
                    break;
                }
            }

            if (!found) {
                JOptionPane.showMessageDialog(this, "Invalid employee number or password.", "Error", JOptionPane.ERROR_MESSAGE);
            }
        } else if (e.getSource() == attendanceButton) {
            // Perform attendance-related actions
            JOptionPane.showMessageDialog(this, "Attendance feature is now Clock In/Out.");
        } else if (e.getSource() == payslipButton) {
            // Generate and display payslip
            if ("admin".equals(currentUserRole)) {
                searchEmployee();
            } else {
                showPayslip(currentEmployeeNumber);
            }
        } else if (e.getSource() == createEntryButton) {
            // Create new employee entry
            createEntry();
        } else if (e.getSource() == deleteEntryButton) {
            // Delete an employee entry
            deleteEntry();
        } else if (e.getSource() == listEmployeesButton) {
            // List all employees
            listEmployees();
        } else if (e.getSource() == searchEmployeeButton) {
            // Search and display employee payslip
            searchEmployee();
        } else if (e.getSource() == viewAttendanceHistoryButton) {
            // View Attendance History
            viewAttendanceHistory();
        } else if (e.getSource() == logoutButton) {
            // Logout action
            logout();
        }
    }

    // Load users from CSV file
    private ArrayList<String[]> loadUsersFromCSV(String filename) {
        ArrayList<String[]> users = new ArrayList<>();
        try (BufferedReader br = new BufferedReader(new FileReader(filename))) {
            String line;
            while ((line = br.readLine()) != null) {
                String[] parts = line.split(",");
                users.add(parts);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return users;
    }

    // Load attendance records from CSV file
    private ArrayList<String[]> loadAttendanceFromCSV(String filename) {
        ArrayList<String[]> records = new ArrayList<>();
        try (BufferedReader br = new BufferedReader(new FileReader(filename))) {
            String line;
            while ((line = br.readLine()) != null) {
                String[] parts = line.split(",");
                records.add(parts);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return records;
    }

    // Display main panel after successful login
    private void displayMainPanel(String employeeName, String employeeNumber) {
        getContentPane().removeAll(); // Remove all components

        JPanel topPanel = new JPanel(new FlowLayout(FlowLayout.LEFT)); // Logout button on the top left
        topPanel.add(logoutButton);
        add(topPanel, BorderLayout.NORTH);

        welcomeLabel.setText("Welcome, " + employeeName + " (Employee Number: " + employeeNumber + ")!");
        mainPanel.add(welcomeLabel, BorderLayout.CENTER);

        if ("admin".equals(currentUserRole)) {
            JPanel adminPanel = new JPanel();
            adminPanel.setLayout(new GridLayout(5, 1)); // Adjusted for five buttons
            adminPanel.add(createEntryButton);
            adminPanel.add(deleteEntryButton);
            adminPanel.add(listEmployeesButton);
            adminPanel.add(searchEmployeeButton);
            adminPanel.add(viewAttendanceHistoryButton);
            mainPanel.add(adminPanel, BorderLayout.EAST);
        } else {
            JPanel userPanel = new JPanel();
            userPanel.setLayout(new GridLayout(4, 1)); // Adjusted for four buttons
            userPanel.add(payslipButton);
            userPanel.add(clockInButton);
            userPanel.add(clockOutButton);
            userPanel.add(viewAttendanceHistoryButton);
            mainPanel.add(userPanel, BorderLayout.EAST);
        }

        add(mainPanel, BorderLayout.CENTER);
        revalidate();
        repaint();
        pack();
    }

    // Create a new employee entry
    private void createEntry() {
        JPanel panel = new JPanel();
        panel.setLayout(new BoxLayout(panel, BoxLayout.Y_AXIS));

        // Scrollable personal information section
        JPanel personalPanel = new JPanel(new GridLayout(0, 2));
        personalPanel.add(new JLabel("Employee Number:"));
        JTextField newEmployeeNumberField = new JTextField(generateNextEmployeeNumber());
        newEmployeeNumberField.setEditable(false);
        personalPanel.add(newEmployeeNumberField);

        personalPanel.add(new JLabel("Last Name:"));
        JTextField newLastNameField = new JTextField();
        personalPanel.add(newLastNameField);

        personalPanel.add(new JLabel("First Name:"));
        JTextField newFirstNameField = new JTextField();
        personalPanel.add(newFirstNameField);

        personalPanel.add(new JLabel("Birthday:"));
        JTextField newBirthdayField = new JTextField();
        personalPanel.add(newBirthdayField);

        personalPanel.add(new JLabel("Address:"));
        JTextField newAddressField = new JTextField();
        personalPanel.add(newAddressField);

        personalPanel.add(new JLabel("Phone Number:"));
        JTextField newPhoneNumberField = new JTextField();
        personalPanel.add(newPhoneNumberField);

        panel.add(personalPanel);

        // Scrollable government IDs section
        JPanel governmentPanel = new JPanel(new GridLayout(0, 2));
        governmentPanel.add(new JLabel("SSS #:"));
        JTextField newSssNumberField = new JTextField();
        governmentPanel.add(newSssNumberField);

        governmentPanel.add(new JLabel("Philhealth #:"));
        JTextField newPhilhealthNumberField = new JTextField();
        governmentPanel.add(newPhilhealthNumberField);

        governmentPanel.add(new JLabel("Pag-IBIG #:"));
        JTextField newPagibigNumberField = new JTextField();
        governmentPanel.add(newPagibigNumberField);

        governmentPanel.add(new JLabel("TIN #:"));
        JTextField newTinNumberField = new JTextField();
        governmentPanel.add(newTinNumberField);

        governmentPanel.add(new JLabel("Department:"));
        JTextField newDepartmentField = new JTextField();
        governmentPanel.add(newDepartmentField);

        governmentPanel.add(new JLabel("Position:"));
        JTextField newPositionField = new JTextField();
        governmentPanel.add(newPositionField);

        governmentPanel.add(new JLabel("Immediate Supervisor:"));
        JTextField newSupervisorField = new JTextField();
        governmentPanel.add(newSupervisorField);

        governmentPanel.add(new JLabel("Basic Salary:"));
        JTextField newSalaryField = new JTextField();
        governmentPanel.add(newSalaryField);
        
        governmentPanel.add(new JLabel("Rice Subsidy:"));
        JTextField newRiceSubsidyField = new JTextField();
        governmentPanel.add(newRiceSubsidyField);

        governmentPanel.add(new JLabel("Phone Allowance:"));
        JTextField newPhoneAllowanceField = new JTextField();
        governmentPanel.add(newPhoneAllowanceField);

        governmentPanel.add(new JLabel("Clothing Allowance:"));
        JTextField newClothingAllowanceField = new JTextField();
        governmentPanel.add(newClothingAllowanceField);

        governmentPanel.add(new JLabel("Semi-Monthly Rate:"));
        JTextField newMonthlyRateField = new JTextField();
        governmentPanel.add(newMonthlyRateField);

        governmentPanel.add(new JLabel("Hourly Rate:"));
        JTextField newHourlyRateField = new JTextField();
        governmentPanel.add(newHourlyRateField);

        panel.add(governmentPanel);
        
        JPanel passwordPanel = new JPanel(new GridLayout(0, 2));
        passwordPanel.add(new JLabel("Password:"));
        JPasswordField newPasswordField = new JPasswordField();
        passwordPanel.add(newPasswordField);
        
        // Role field with dropdown options
    	JPanel rolePanel = new JPanel(new GridLayout(0, 2));
    	rolePanel.add(new JLabel("Role:"));
    	JComboBox<String> roleComboBox = new JComboBox<>(new String[]{"admin", "user"});
    	rolePanel.add(roleComboBox);

        panel.add(passwordPanel);
        panel.add(rolePanel);

        JScrollPane scrollPane = new JScrollPane(panel);
        scrollPane.setPreferredSize(new Dimension(500, 400)); // Adjust size as needed

        // Display the dialog box
        int option = JOptionPane.showConfirmDialog(this, panel, "Create New Employee Entry", JOptionPane.OK_CANCEL_OPTION, JOptionPane.PLAIN_MESSAGE);

        if (option == JOptionPane.OK_OPTION) {
            // Construct the user data array
            String[] userData = {
                    newEmployeeNumberField.getText(),
                    newLastNameField.getText(),
                    newFirstNameField.getText(),
                    newBirthdayField.getText(),
                    newAddressField.getText(),
                    newPhoneNumberField.getText(),
                    newSssNumberField.getText(),
                    newPhilhealthNumberField.getText(),
                    newPagibigNumberField.getText(),
                    newTinNumberField.getText(),
                    newDepartmentField.getText(),
                    newPositionField.getText(),
                    newSupervisorField.getText(),
                    newSalaryField.getText(),
                    newRiceSubsidyField.getText(),
                newPhoneAllowanceField.getText(),
                newClothingAllowanceField.getText(),
                newMonthlyRateField.getText(),
                newHourlyRateField.getText(),
                new String(newPasswordField.getPassword()),
                (String) roleComboBox.getSelectedItem() 
            };

            users.add(userData);
            saveUsersToCSV("users.csv", users);
            JOptionPane.showMessageDialog(this, "Employee created successfully.");
        }
    }
    
    // Save users to CSV file
    private void saveUsersToCSV(String filename, ArrayList<String[]> users) {
        try (BufferedWriter bw = new BufferedWriter(new FileWriter(filename))) {
            for (String[] user : users) {
                bw.write(String.join(",", user));
                bw.newLine();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

// Generate next employee number (assuming numeric)
    private String generateNextEmployeeNumber() {
        int max = 0;
        for (String[] user : users) {
            try {
                int currentNumber = Integer.parseInt(user[0]);
                if (currentNumber > max) {
                    max = currentNumber;
                }
            } catch (NumberFormatException e) {
                // Ignore non-numeric employee numbers
            }
        }
        return String.valueOf(max + 1);
    }


    // Delete an employee entry
    private void deleteEntry() {
        String employeeNumber = JOptionPane.showInputDialog(this, "Enter employee number to delete:");
        if (employeeNumber != null && !employeeNumber.isEmpty()) {
            boolean found = false;
            for (Iterator<String[]> iterator = users.iterator(); iterator.hasNext(); ) {
                String[] user = iterator.next();
                if (user[0].equals(employeeNumber)) {
                    iterator.remove();
                    found = true;
                    saveUsersToCSV("users.csv", users);
                    JOptionPane.showMessageDialog(this, "Employee deleted successfully.");
                    break;
                }
            }
            if (!found) {
                JOptionPane.showMessageDialog(this, "Employee number not found.", "Error", JOptionPane.ERROR_MESSAGE);
            }
        }
    }

    // **Updated** List all employees with a searchable GUI
    private void listEmployees() {
        JFrame listFrame = new JFrame("List Employees");
        listFrame.setSize(800, 600);
        listFrame.setLocationRelativeTo(null);

        String[] columnNames = {
                "Employee Number", "Last Name", "First Name",
                "Birthday", "Address", "Phone Number",
                "SSS #", "Philhealth #", "Pag-IBIG #", "TIN #",
                "Department", "Position", "Immediate Supervisor",
                "Basic Salary", "Rice Subsidy", "Phone Allowance",
                "Clothing Allowance", "Semi-Monthly Rate", "Hourly Rate"
        };

        // Convert users list to Object[][] for the JTable
        Object[][] data = new Object[users.size()][columnNames.length];
        for (int i = 0; i < users.size(); i++) {
            data[i] = users.get(i);
        }

        DefaultTableModel model = new DefaultTableModel(data, columnNames);
        JTable table = new JTable(model);

        JScrollPane scrollPane = new JScrollPane(table);

        JPanel searchPanel = new JPanel(new BorderLayout());
        JTextField searchField = new JTextField();
        JButton searchButton = new JButton("Search");

        searchButton.addActionListener(e -> {
            String searchText = searchField.getText().toLowerCase();
            TableRowSorter<DefaultTableModel> sorter = new TableRowSorter<>(model);
            table.setRowSorter(sorter);
            if (searchText.length() == 0) {
                sorter.setRowFilter(null);
            } else {
                sorter.setRowFilter(RowFilter.regexFilter("(?i)" + searchText));
            }
        });

        searchPanel.add(searchField, BorderLayout.CENTER);
        searchPanel.add(searchButton, BorderLayout.EAST);

        listFrame.setLayout(new BorderLayout());
        listFrame.add(searchPanel, BorderLayout.NORTH);
        listFrame.add(scrollPane, BorderLayout.CENTER);

        listFrame.setVisible(true);
    }

    // Search for an employee by employee number
    private void searchEmployee() {
        String employeeNumber = JOptionPane.showInputDialog(this, "Enter Employee Number to Search:");

        if (employeeNumber != null && !employeeNumber.isEmpty()) {
            boolean found = false;
            for (String[] user : users) {
                if (user[0].equals(employeeNumber)) {
                    showPayslip(employeeNumber);
                    found = true;
                    break;
                }
            }

            if (!found) {
                JOptionPane.showMessageDialog(this, "Employee number not found.", "Error", JOptionPane.ERROR_MESSAGE);
            }
        }
    }

    // Show payslip for the given employee number
    private void showPayslip(String employeeNumber) {
        String[] columnNames = {
                "Earnings/Deductions", "Amount"
        };

        Object[][] data = {
                {"Basic Salary", "$1500.00"},
                {"Overtime", "$200.00"},
                {"Total Earnings", "$1700.00"},
                {"Tax", "$150.00"},
                {"SSS", "$50.00"},
                {"Philhealth", "$100.00"},
                {"Pag-IBIG", "$50.00"},
                {"Total Deductions", "$350.00"},
                {"Net Salary", "$1350.00"}
        };

        JTable table = new JTable(new DefaultTableModel(data, columnNames));
        JOptionPane.showMessageDialog(this, new JScrollPane(table), "Payslip for Employee " + employeeNumber, JOptionPane.PLAIN_MESSAGE);
    }

    // Clock in for attendance
    private void clockIn() {
        // Record clock-in time
        String[] record = {currentEmployeeNumber, "Clock In", new Date().toString()};
        attendanceRecords.add(record);
        saveAttendanceToCSV("attendance.csv", attendanceRecords);
        JOptionPane.showMessageDialog(this, "Clocked in successfully.");
    }

    // Clock out for attendance
    private void clockOut() {
        // Record clock-out time
        String[] record = {currentEmployeeNumber, "Clock Out", new Date().toString()};
        attendanceRecords.add(record);
        saveAttendanceToCSV("attendance.csv", attendanceRecords);
        JOptionPane.showMessageDialog(this, "Clocked out successfully.");
    }

    // View Attendance History
    private void viewAttendanceHistory() {
        if ("admin".equals(currentUserRole)) {
            String employeeNumber = JOptionPane.showInputDialog(this, "Enter employee number to view attendance:");
            if (employeeNumber != null && !employeeNumber.isEmpty()) {
                displayAttendanceHistory(employeeNumber);
            }
        } else {
            displayAttendanceHistory(currentEmployeeNumber);
        }
    }

    // Display attendance history
    private void displayAttendanceHistory(String employeeNumber) {
        StringBuilder history = new StringBuilder("Attendance History:\n");
        for (String[] record : attendanceRecords) {
            if (record[0].equals(employeeNumber)) {
                history.append(String.format("Date: %s, Action: %s\n", record[2], record[1]));
            }
        }
        JOptionPane.showMessageDialog(this, history.toString());
    }

    // Save attendance records to CSV
    private void saveAttendanceToCSV(String filename, ArrayList<String[]> attendanceRecords) {
        try (BufferedWriter bw = new BufferedWriter(new FileWriter(filename))) {
            for (String[] record : attendanceRecords) {
                bw.write(String.join(",", record));
                bw.newLine();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // Logout current user
    private void logout() {
        getContentPane().removeAll(); // Remove all components
        getContentPane().setLayout(new GridLayout(3, 2));

        welcomeLabel.setText("");
        mainPanel.removeAll();

        // Reset login panel
        employeeNumberField.setText("");
        passwordField.setText("");

        add(employeeNumberLabel);
        add(employeeNumberField);

        add(passwordLabel);
        add(passwordField);

        add(loginButton);

        revalidate();
        repaint();
        pack();
    }

    // Main method to start the application
    public static void main(String[] args) {
        SwingUtilities.invokeLater(new Runnable() {
            public void run() {
                new EmployeeApplication().setVisible(true);
            }
        });
    }
}
