# ToDoListapp
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.io.*;
import javax.swing.event.DocumentListener;
import javax.swing.event.DocumentEvent;
import java.util.*;

public class ToDoListApp {
    private static final String FILE_NAME = "tasks.txt";
    private DefaultListModel<Task> taskListModel;
    private JList<Task> taskList;
    private boolean isDarkMode = false;
    private JFrame frame;
    private JTextField TaskField;
    private JButton AddButton;
    private JButton RemoveButton;
    private JButton ClearButton;
    private JButton DarkModeButton;

    public ToDoListApp() {
        frame = new JFrame("To-Do List");
        frame.setSize(400, 400);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setLayout(new BorderLayout());

        setupTaskList();
        frame.add(new JScrollPane(taskList), BorderLayout.CENTER);

        setupInputActions();
        frame.add(setupButtonPanel(), BorderLayout.SOUTH);

        setupSearchBar();
        frame.setVisible(true);

        setupKeyboardShortcuts();
    }

    private void setupTaskList() {
        taskListModel = new DefaultListModel<>();
        taskList = new JList<>(taskListModel);
        taskList.setCellRenderer(new TaskCellRenderer());
        taskList.setSelectionMode(ListSelectionModel.SINGLE_SELECTION);
        loadTasks();
    }

    private void setupInputActions() {
        TaskField = new JTextField(20);
        AddButton = new JButton("Add");
        RemoveButton = new JButton("Remove");
        ClearButton = new JButton("Clear All");
        DarkModeButton = new JButton("Dark Mode");

        AddButton.addActionListener(e -> addTask());
        RemoveButton.addActionListener(e -> removeTask());
        ClearButton.addActionListener(e -> clearAllTasks());
        DarkModeButton.addActionListener(e -> toggleDarkMode());
    }

    private JPanel setupButtonPanel() {
        JPanel panel = new JPanel(new GridLayout(2, 2, 5, 5));
        panel.add(new JLabel("Task:"));
        panel.add(TaskField);
        panel.add(AddButton);
        panel.add(RemoveButton);
        panel.add(ClearButton);
        panel.add(DarkModeButton);
        return panel;
    }

    private void addTask() {
        String taskText = TaskField.getText().trim();
        if (!taskText.isEmpty()) {
            taskListModel.addElement(new Task(taskText));
            TaskField.setText("");
            saveTasks();
        }
    }

    private void removeTask() {
        int selectedIndex = taskList.getSelectedIndex();
        if (selectedIndex != -1) {
            taskListModel.remove(selectedIndex);
            saveTasks();
        } else {
            JOptionPane.showMessageDialog(frame, "Select a task to remove.", "No Task Selected",
                    JOptionPane.WARNING_MESSAGE);
        }
    }

    private void clearAllTasks() {
        int response = JOptionPane.showConfirmDialog(frame, "Clear all tasks?", "Confirm Clear",
                JOptionPane.YES_NO_OPTION, JOptionPane.WARNING_MESSAGE);
        if (response == JOptionPane.YES_OPTION) {
            taskListModel.clear();
            saveTasks();
        }
    }

    private void toggleDarkMode() {
        isDarkMode = !isDarkMode;
        updateTheme();
    }

    private void loadTasks() {
        try (BufferedReader reader = new BufferedReader(new FileReader(FILE_NAME))) {
            String line;
            while ((line = reader.readLine()) != null) {
                String[] parts = line.split("\\|");
                taskListModel.addElement(new Task(parts[0], Boolean.parseBoolean(parts[1])));
            }
        } catch (IOException e) {
            System.out.println("No previous tasks found.");
        }
    }

    private void saveTasks() {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(FILE_NAME))) {
            for (int i = 0; i < taskListModel.size(); i++) {
                Task task = taskListModel.get(i);
                writer.write(task.getText() + "|" + task.isCompleted());
                writer.newLine();
            }
        } catch (IOException e) {
            System.out.println("Error saving tasks.");
        }
    }

    private void setupSearchBar() {
        JTextField searchField = new JTextField(20);
        frame.add(searchField, BorderLayout.NORTH);
        searchField.getDocument().addDocumentListener(new DocumentListener() {
            @Override
            public void insertUpdate(DocumentEvent e) {
                filterTasks(searchField.getText());
            }

            @Override
            public void removeUpdate(DocumentEvent e) {
                filterTasks(searchField.getText());
            }

            @Override
            public void changedUpdate(DocumentEvent e) {
                filterTasks(searchField.getText());
            }
        });
    }

    private void filterTasks(String query) {
        DefaultListModel<Task> filteredModel = new DefaultListModel<>();
        for (int i = 0; i < taskListModel.size(); i++) {
            Task task = taskListModel.get(i);
            if (task.getText().toLowerCase().contains(query.toLowerCase())) {
                filteredModel.addElement(task);
            }
        }
        taskList.setModel(filteredModel);
    }

    private void updateTheme() {
        Color background = isDarkMode ? Color.DARK_GRAY : Color.WHITE;
        Color foreground = isDarkMode ? Color.WHITE : Color.BLACK;

        frame.getContentPane().setBackground(background);
        taskList.setBackground(isDarkMode ? Color.BLACK : Color.WHITE);
        taskList.setForeground(foreground);
        if (isDarkMode) {
            AddButton.setBackground(Color.GRAY);
            RemoveButton.setBackground(Color.GRAY);
            ClearButton.setBackground(Color.GRAY);
            DarkModeButton.setBackground(Color.GRAY);
            AddButton.setForeground(Color.WHITE);
            RemoveButton.setForeground(Color.WHITE);
            ClearButton.setForeground(Color.WHITE);
            DarkModeButton.setForeground(Color.WHITE);
            TaskField.setBackground(Color.BLACK);
            TaskField.setForeground(Color.WHITE);
        } else {
            AddButton.setBackground(UIManager.getColor("Button.background"));
            RemoveButton.setBackground(UIManager.getColor("Button.background"));
            ClearButton.setBackground(UIManager.getColor("Button.background"));
            DarkModeButton.setBackground(UIManager.getColor("Button.background"));
            AddButton.setForeground(UIManager.getColor("Button.foreground"));
            RemoveButton.setForeground(UIManager.getColor("Button.foreground"));
            ClearButton.setForeground(UIManager.getColor("Button.foreground"));
            DarkModeButton.setForeground(UIManager.getColor("Button.foreground"));
            TaskField.setBackground(Color.WHITE);
            TaskField.setForeground(Color.BLACK);
        }
    }

    private void setupKeyboardShortcuts() {
        TaskField.addKeyListener(new KeyAdapter() {
            @Override
            public void keyPressed(KeyEvent e) {
                if (e.getKeyCode() == KeyEvent.VK_ENTER) {
                    AddButton.doClick();
                } else if (e.getKeyCode() == KeyEvent.VK_DELETE) {
                    RemoveButton.doClick();
                }
            }
        });
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(ToDoListApp::new);
    }

    static class Task {
        private String text;
        private boolean completed;

        public Task(String text) {
            this(text, false);
        }

        public Task(String text, boolean completed) {
            this.text = text;
            this.completed = completed;
        }

        public String getText() {
            return text;
        }

        public boolean isCompleted() {
            return completed;
        }

        public void toggleCompletion() {
            this.completed = !completed;
        }

        @Override
        public String toString() {
            return text + (completed ? " (completed)" : "");
        }
    }

    static class TaskCellRenderer extends JLabel implements ListCellRenderer<Task> {
        @Override
        public Component getListCellRendererComponent(JList<? extends Task> list, Task value, int index,
                boolean isSelected, boolean cellHasFocus) {
            setText(value.getText());
            setFont(getFont().deriveFont(value.isCompleted() ? Font.ITALIC : Font.PLAIN));
            setForeground(isSelected ? Color.YELLOW : list.getForeground()); 
            setBackground(isSelected ? list.getSelectionBackground() : list.getBackground()); 
            if (value.isCompleted()) {
                setText("<html><strike>" + value.getText() + "</strike></html>");
            }
            return this;
        }
    }
}
