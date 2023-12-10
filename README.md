import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.util.HashMap;
import java.util.Map;
import java.util.Random;
import java.util.Timer;
import java.util.TimerTask;

public class StockSimulator {
    private static Map<String, Double> stockData = new HashMap<>();
    private static Map<String, JTextField> priceFields = new HashMap<>();
    private static Map<String, Integer> stocksBought = new HashMap<>();
    private static double balance = 10000.0; // Initial balance

    private static JFrame frame;
    private static JTextArea stocksTextArea;
    private static JLabel balanceLabel;

    public static void main(String[] args) {
        frame = new JFrame("Stock Simulator");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setSize(1200, 800);
        frame.setLayout(new BorderLayout());

        // Create colorful panels
        JPanel stockInfoPanel = new JPanel(new GridLayout(1, 3));
        stockInfoPanel.setBorder(BorderFactory.createTitledBorder("STOCK INFORMATION"));
        stockInfoPanel.setBackground(new Color(200, 230, 200)); // Green background

        String[] categories = {"Currency Stocks", "Economy Stocks", "Company Stocks"};
        for (String category : categories) {
            JPanel stockCategoryPanel = createStockPanel(category);
            stockInfoPanel.add(stockCategoryPanel);
            String[] stockNames = getStockNamesForCategory(category);
            for (String stockName : stockNames) {
                addStockInfoLabelsAndFields(stockCategoryPanel, stockName);
            }
        }

        // Create colorful button panel
        JPanel buttonPanel = new JPanel(new FlowLayout(FlowLayout.CENTER));
        buttonPanel.setBackground(new Color(220, 220, 240)); // Light blue background
        JButton buySellButton = createStyledButton("Buy/Sell Stock", Color.BLUE);
        JButton viewGraphButton = createStyledButton("View Graph", Color.ORANGE);
        buttonPanel.add(buySellButton);
        buttonPanel.add(viewGraphButton);

        // Create colorful stocks bought panel
        JPanel stocksBoughtPanel = new JPanel(new BorderLayout());
        stocksBoughtPanel.setBorder(BorderFactory.createTitledBorder("Stocks Bought"));
        stocksBoughtPanel.setBackground(new Color(240, 200, 200)); // Light red background
        stocksTextArea = new JTextArea("STOCK NAME - BUY PRICE/SELL PRICE - Quantity\n");
        stocksTextArea.setEditable(false);
        JScrollPane stocksScrollPane = new JScrollPane(stocksTextArea);
        balanceLabel = new JLabel("BALANCE: $" + String.format("%.2f", balance));
        stocksBoughtPanel.add(stocksScrollPane, BorderLayout.CENTER);
        stocksBoughtPanel.add(balanceLabel, BorderLayout.SOUTH);

        frame.add(stockInfoPanel, BorderLayout.CENTER);
        frame.add(buttonPanel, BorderLayout.SOUTH);
        frame.add(stocksBoughtPanel, BorderLayout.EAST);

        frame.setVisible(true);

        buySellButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                showBuySellDialog();
            }
        });

        viewGraphButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                showViewGraphDialog();
            }
        });

        initializeStockData();

        Timer timer = new Timer();
        timer.scheduleAtFixedRate(new TimerTask() {
            @Override
            public void run() {
                updateStockValues();
            }
        }, 0, 2000);
    }

    private static JPanel createStockPanel(String title) {
        JPanel stockPanel = new JPanel(new GridLayout(0, 2));
        stockPanel.setBorder(BorderFactory.createTitledBorder(title));
        stockPanel.setBackground(new Color(220, 240, 220)); // Light green background
        return stockPanel;
    }

    private static void addStockInfoLabelsAndFields(JPanel stockPanel, String stockName) {
        JLabel nameLabel = new JLabel(stockName);
        JTextField changeField = new JTextField();
        changeField.setEditable(false);
        changeField.setBackground(new Color(240, 240, 240)); // Light gray background
        priceFields.put(stockName, changeField);
        stockPanel.add(nameLabel);
        stockPanel.add(changeField);
    }

    private static void updateStockValues() {
        for (String stockName : stockData.keySet()) {
            double currentPrice = stockData.get(stockName);
            double priceChange = (Math.random() - 0.5) * 10; // Random price change between -5 and 5
            double newPrice = currentPrice + priceChange;
            stockData.put(stockName, newPrice);

            SwingUtilities.invokeLater(new Runnable() {
                @Override
                public void run() {
                    priceFields.get(stockName).setText(String.format("%.2f", newPrice));
                }
            });
        }
    }

    private static String[] getStockNamesForCategory(String category) {
        switch (category) {
            case "Currency Stocks":
                return new String[]{"EUR/USD", "GBP/USD", "EUR/GBP", "GBP/JPY"};
            case "Economy Stocks":
                return new String[]{"FTSE 100", "Dow Jones", "$AUSSIE200", "NIKKEI225"};
            case "Company Stocks":
                return new String[]{"Facebook", "Apple", "Microsoft", "BMW"};
            default:
                return new String[]{};
        }
    }

    private static void initializeStockData() {
        String[] allStockNames = getAllStockNames();
        for (String stockName : allStockNames) {
            double randomPrice = 100.0 + (new Random().nextDouble() * 100.0);
            stockData.put(stockName, randomPrice);
        }
    }

    private static String[] getAllStockNames() {
        return new String[]{
                "EUR/USD", "GBP/USD", "EUR/GBP", "GBP/JPY",
                "FTSE 100", "Dow Jones", "$AUSSIE200", "NIKKEI225",
                "Facebook", "Apple", "Microsoft", "BMW"
        };
    }

    private static void showBuySellDialog() {
        String[] shareNames = getAllStockNames();
        JComboBox<String> shareNameComboBox = new JComboBox<>(shareNames);
        JTextField quantityTextField = new JTextField();
        JRadioButton buyButton = new JRadioButton("Buy");
        JRadioButton sellButton = new JRadioButton("Sell");
        ButtonGroup buttonGroup = new ButtonGroup();
        buttonGroup.add(buyButton);
        buttonGroup.add(sellButton);

        JPanel dialogPanel = new JPanel(new GridLayout(4, 2));
        dialogPanel.add(new JLabel("Share Name:"));
        dialogPanel.add(shareNameComboBox);
        dialogPanel.add(new JLabel("Quantity:"));
        dialogPanel.add(quantityTextField);
        dialogPanel.add(buyButton);
        dialogPanel.add(sellButton);

        int option = JOptionPane.showConfirmDialog(frame, dialogPanel, "Buy/Sell Stock", JOptionPane.OK_CANCEL_OPTION);
        if (option == JOptionPane.OK_OPTION) {
            String selectedStock = shareNameComboBox.getSelectedItem().toString();
            String quantityString = quantityTextField.getText();

            try {
                int quantity = Integer.parseInt(quantityString);
                if (quantity > 0) {
                    if (buyButton.isSelected()) {
                        buyStock(selectedStock, quantity);
                    } else if (sellButton.isSelected()) {
                        sellStock(selectedStock, quantity);
                    }
                } else {
                    JOptionPane.showMessageDialog(frame, "Invalid quantity input. Please enter a valid number.");
                }
            } catch (NumberFormatException ex) {
                JOptionPane.showMessageDialog(frame, "Invalid quantity input. Please enter a valid number.");
            }
        }
    }

    private static void showViewGraphDialog() {
        String[] shareNames = getAllStockNames();
        JComboBox<String> shareNameComboBox = new JComboBox<>(shareNames);
    
        JPanel dialogPanel = new JPanel(new GridLayout(2, 2));
        dialogPanel.add(new JLabel("Share Name:"));
        dialogPanel.add(shareNameComboBox);
    
        int option = JOptionPane.showConfirmDialog(frame, dialogPanel, "View Graph", JOptionPane.OK_CANCEL_OPTION);
        if (option == JOptionPane.OK_OPTION) {
            String selectedStock = shareNameComboBox.getSelectedItem().toString();
            
            if (selectedStock.equals("EUR/USD")) {
                // Display the image for "EUR/USD"
                ImageIcon icon = new ImageIcon("C:\\Users\\prath\\OneDrive\\Desktop\\EUR- USD Change.jpg");
                JLabel imageLabel = new JLabel(icon);
                JOptionPane.showMessageDialog(frame, imageLabel, "EUR/USD Graph", JOptionPane.PLAIN_MESSAGE);
            } else {
                // Handle other stocks
                // You can implement logic to display graphs or relevant information for other stocks.
            }
        }
    }
    

    private static void buyStock(String stockName, int quantity) {
        if (stockData.containsKey(stockName)) {
            double stockPrice = stockData.get(stockName);
            double totalCost = stockPrice * quantity;
            if (balance >= totalCost) {
                balance -= totalCost;
                updateStocksBought(stockName, "Buy Quantity", quantity, stockPrice);
            } else {
                JOptionPane.showMessageDialog(frame, "Insufficient balance. You cannot buy this stock.");
            }
        }
    }

    private static void sellStock(String stockName, int quantity) {
        if (stockData.containsKey(stockName) && stocksBought.containsKey(stockName)) {
            int availableQuantity = stocksBought.get(stockName);
            if (availableQuantity >= quantity) {
                double stockPrice = stockData.get(stockName);
                double totalAmount = stockPrice * quantity;
                balance += totalAmount;
                updateStocksBought(stockName, "Sell Quantity", quantity, stockPrice);
            } else {
                JOptionPane.showMessageDialog(frame, "You don't have enough stocks to sell.");
            }
        }
    }

    private static void updateStocksBought(String shareName, String transactionType, int quantity, double price) {
        String currentText = stocksTextArea.getText();

        if (transactionType.equals("Buy Quantity")) {
            stocksBought.put(shareName, stocksBought.getOrDefault(shareName, 0) + quantity);
            currentText += "\n" + shareName + " - Buy Price: $" + String.format("%.2f", price) + " - Quantity: " + quantity;
        } else if (transactionType.equals("Sell Quantity")) {
            int currentQuantity = stocksBought.getOrDefault(shareName, 0);
            if (currentQuantity >= quantity) {
                stocksBought.put(shareName, currentQuantity - quantity);
                double totalAmount = price * quantity;
                currentText += "\n" + shareName + " - Sell Price: $" + String.format("%.2f", price) + " - Quantity: " + quantity + " - Total Amount: $" + String.format("%.2f", totalAmount);
            } else {
                JOptionPane.showMessageDialog(frame, "You don't have enough stocks to sell.");
            }
        }

        stocksTextArea.setText(currentText);
        balanceLabel.setText("BALANCE: $" + String.format("%.2f", balance));
    }

    private static JButton createStyledButton(String text, Color color) {
        JButton button = new JButton(text);
        button.setBackground(color);
        button.setForeground(Color.WHITE);
        button.setBorderPainted(false);
        button.setFocusPainted(false);
        button.setOpaque(true);
        return button;
    }
}
