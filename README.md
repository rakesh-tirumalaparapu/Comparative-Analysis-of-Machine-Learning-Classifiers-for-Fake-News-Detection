# Comparative-Analysis-of-Machine-Learning-Classifiers-for-Fake-News-Detection


public class CustomerDAOImpl implements CustomerDAO {

    // ... existing fields ...
    private final String url="jdbc:postgresql://localhost:5412/postgres";
    private final String username="postgres";
    private final String password="Vishnu@@3168"; // ⚠️ Hardcoding credentials is bad practice!

    // ... existing addCustomer method ...
    @Override
    public String addCustomer(Customer customer) {
        // ... existing implementation ...
        try {
            Connection connection = DriverManager.getConnection(url, username, password);
            String sqlQuery = "insert into customer values (?,?,?,?,?)"; // Assuming 5 fields
            PreparedStatement preparedStatement = connection.prepareStatement(sqlQuery);
            preparedStatement.setInt(1, customer.getCustomerId());
            preparedStatement.setString(2, customer.getName());
            preparedStatement.setString(3, customer.getContact());
            preparedStatement.setString(4, customer.getAccountType());
            preparedStatement.setString(5, customer.getEmail());

            int rowCount = preparedStatement.executeUpdate();

            if (rowCount > 0) {
                return "Customer added successfully";
            }
            // Close resources
            preparedStatement.close();
            connection.close();

        } catch (SQLException e) {
            e.printStackTrace(); // Log the error
            throw new RuntimeException(e);
        }
        return "Unable to add Customer";
    }

    // --- Search Operation (findCustomerById) ---
    @Override
    public Customer findCustomerById(Integer id) throws CustomerNotFoundException {
        // ... previous placeholder logic: /* for(Customer customer : customers) { ... } */ ...

        try (Connection connection = DriverManager.getConnection(url, username, password);
             PreparedStatement preparedStatement = connection.prepareStatement("SELECT * FROM customer WHERE customerid = ?")) {

            preparedStatement.setInt(1, id);
            ResultSet rs = preparedStatement.executeQuery();

            if (rs.next()) {
                // Assuming Customer class has a constructor or setters for these fields
                Customer customer = new Customer();
                customer.setCustomerId(rs.getInt("customerid"));
                customer.setName(rs.getString("name"));
                customer.setContact(rs.getString("contact"));
                customer.setAccountType(rs.getString("accounttype"));
                customer.setEmail(rs.getString("email"));
                
                rs.close();
                return customer;
            }
            rs.close();
        } catch (SQLException e) {
            e.printStackTrace();
            throw new RuntimeException(e);
        }

        // Throw exception if no customer is found
        throw new CustomerNotFoundException("Customer with id " + id + " not found");
    }

    // --- Update Operation (updateCustomer) ---
    @Override
    public Customer updateCustomer(Customer customer) throws CustomerNotFoundException {
        // return null; // Original placeholder
        try (Connection connection = DriverManager.getConnection(url, username, password);
             PreparedStatement preparedStatement = connection.prepareStatement(
                 "UPDATE customer SET name=?, contact=?, accounttype=?, email=? WHERE customerid=?"
             )) {

            preparedStatement.setString(1, customer.getName());
            preparedStatement.setString(2, customer.getContact());
            preparedStatement.setString(3, customer.getAccountType());
            preparedStatement.setString(4, customer.getEmail());
            preparedStatement.setInt(5, customer.getCustomerId());

            int rowCount = preparedStatement.executeUpdate();

            if (rowCount > 0) {
                return customer; // Return the updated customer object
            }
        } catch (SQLException e) {
            e.printStackTrace();
            throw new RuntimeException(e);
        }

        // Throw exception if no customer was updated (i.e., not found)
        throw new CustomerNotFoundException("Customer with id " + customer.getCustomerId() + " not found for update.");
    }

    // --- Delete Operation (deleteCustomer) ---
    @Override
    public String deleteCustomer(Integer id) throws CustomerNotFoundException {
        // return ""; // Original placeholder
        try (Connection connection = DriverManager.getConnection(url, username, password);
             PreparedStatement preparedStatement = connection.prepareStatement("DELETE FROM customer WHERE customerid = ?")) {

            preparedStatement.setInt(1, id);
            int rowCount = preparedStatement.executeUpdate();

            if (rowCount > 0) {
                return "Customer with id " + id + " deleted successfully.";
            }
        } catch (SQLException e) {
            e.printStackTrace();
            throw new RuntimeException(e);
        }

        // Throw exception if no customer was deleted (i.e., not found)
        throw new CustomerNotFoundException("Customer with id " + id + " not found for deletion.");
    }
}


public class CustomerServiceImpl implements CustomerService {

    CustomerDAO customerDAO = new CustomerDAOImpl();

    @Override
    public String addCustomer(Customer customer) {
        return customerDAO.addCustomer(customer);
    }

    @Override
    public Customer findCustomerById(Integer id) throws CustomerNotFoundException {
        return customerDAO.findCustomerById(id);
    }

    @Override
    public Customer updateCustomer(Customer customer) throws CustomerNotFoundException {
        return customerDAO.updateCustomer(customer);
    }

    @Override
    public String deleteCustomer(Integer id) throws CustomerNotFoundException {
        return customerDAO.deleteCustomer(id);
    }
}

import java.util.Scanner;
// Assuming all necessary classes are imported

public class Demo {
    public static void main(String[] args) {
        CustomerService service = new CustomerServiceImpl();
        Scanner sc = new Scanner(System.in);

        while(true) {
            System.out.println("\nWelcome to Standard Chartered Bank");
            System.out.println("Please enter your choice");
            System.out.println("1 for Add New Customer");
            System.out.println("2 for Display Customer"); // This will be used for Search by ID
            System.out.println("3 for Search Customer"); // Let's use 2 and 3 for the same function, Search by ID
            System.out.println("4 for Delete Customer");
            System.out.println("5 for Exit the bank application");

            // Assuming a method for a simple display of all customers is not required by the prompt
            // and that "Display Customer" should logically be a Search by ID.
            
            Integer choice = sc.nextInt();
            // Consume the newline character left by nextInt()
            sc.nextLine(); 

            switch (choice) {
                case 1: // Add New Customer
                    System.out.println("Please enter customer details :");
                    Scanner sc1 = new Scanner(System.in);
                    Customer customer = new Customer();

                    System.out.print("Enter customerID : ");
                    Integer customerID = sc1.nextInt();
                    sc1.nextLine(); // Consume newline

                    System.out.print("Enter name : ");
                    String name = sc1.nextLine();

                    System.out.print("Enter email : ");
                    String email = sc1.nextLine();
                    
                    System.out.print("Enter contact : ");
                    String contact = sc1.nextLine();
                    
                    System.out.print("Enter account type (Savings or Current) : ");
                    String accountType = sc1.nextLine();

                    customer.setCustomerId(customerID);
                    customer.setName(name);
                    customer.setContact(contact);
                    customer.setEmail(email);
                    customer.setAccountType(accountType);
                    
                    String message = service.addCustomer(customer);
                    System.out.println(message);
                    break;
                
                case 2: // Display/Search Customer by ID (Assuming this is the intended function)
                case 3: // Search Customer by ID
                    System.out.print("Enter Customer ID to search: ");
                    Integer searchId = sc.nextInt();
                    sc.nextLine(); // Consume newline
                    try {
                        Customer foundCustomer = service.findCustomerById(searchId);
                        System.out.println("\n--- Customer Details Found ---");
                        System.out.println("ID: " + foundCustomer.getCustomerId());
                        System.out.println("Name: " + foundCustomer.getName());
                        System.out.println("Contact: " + foundCustomer.getContact());
                        System.out.println("Email: " + foundCustomer.getEmail());
                        System.out.println("Account Type: " + foundCustomer.getAccountType());
                        System.out.println("----------------------------\n");
                    } catch (CustomerNotFoundException e) {
                        System.out.println(e.getMessage());
                    } catch (Exception e) {
                        System.out.println("An error occurred during search: " + e.getMessage());
                    }
                    break;
                
                case 4: // Delete Customer
                    System.out.print("Enter Customer ID to delete: ");
                    Integer deleteId = sc.nextInt();
                    sc.nextLine(); // Consume newline
                    try {
                        String deleteMessage = service.deleteCustomer(deleteId);
                        System.out.println(deleteMessage);
                    } catch (CustomerNotFoundException e) {
                        System.out.println(e.getMessage());
                    } catch (Exception e) {
                        System.out.println("An error occurred during deletion: " + e.getMessage());
                    }
                    break;
                
                case 5: // Exit
                    System.out.println("Exiting the bank application. Goodbye!");
                    System.exit(0);
                    
                default:
                    System.out.println("Invalid choice. Please enter a number between 1 and 5.");
            }
        }
    }
}





