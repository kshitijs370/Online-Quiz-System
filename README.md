# Online Quiz System
import java.io.*;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

// 1. Question Class to represent individual quiz questions
class Question {
    private String questionText;
    private List<String> options;
    private int correctAnswerIndex; // 0-based index

    public Question(String questionText, List<String> options, int correctAnswerIndex) {
        this.questionText = questionText;
        this.options = options;
        this.correctAnswerIndex = correctAnswerIndex;
    }

    public String getQuestionText() { return questionText; }
    public List<String> getOptions() { return options; }
    public int getCorrectAnswerIndex() { return correctAnswerIndex; }

    // Helper method to serialize a question into a single text line for file storage
    public String toFileString() {
        return questionText + "::" + String.join("##", options) + "::" + correctAnswerIndex;
    }

    // Helper method to recreate a question object from a text line
    public static Question fromFileString(String line) {
        String[] parts = line.split("::");
        if (parts.length < 3) return null;
        String text = parts[0];
        List<String> options = List.of(parts[1].split("##"));
        int correctIndex = Integer.parseInt(parts[2]);
        return new Question(text, options, correctIndex);
    }
}

// 2. Main System Class
public class QuizSystem {
    private static List<Question> quizRepository = new ArrayList<>();
    private static final Scanner scanner = new Scanner(System.in);
    
    // File names for persistence
    private static final String QUESTIONS_FILE = "questions.txt";
    private static final String PERFORMANCE_FILE = "performance.txt";

    public static void main(String[] args) {
        // Load questions from file on start
        loadQuestionsFromFile();

        System.out.println("=== WELCOME TO THE ONLINE QUIZ SYSTEM ===");
        
        while (true) {
            System.out.println("\nLogin As:");
            System.out.println("1. Teacher");
            System.out.println("2. Student");
            System.out.println("3. Exit");
            System.out.print("Choose an option: ");
            
            int choice = getValidIntegerInput();

            switch (choice) {
                case 1:
                    teacherLogin();
                    break;
                case 2:
                    studentLogin();
                    break;
                case 3:
                    System.out.println("Thank you for using the Quiz System. Goodbye!");
                    System.exit(0);
                default:
                    System.out.println("Invalid choice! Please select 1, 2, or 3.");
            }
        }
    }

    // --- FILE HANDLING METHODS ---
    
    private static void loadQuestionsFromFile() {
        File file = new File(QUESTIONS_FILE);
        if (!file.exists()) {
            // Seed initial data if the file doesn't exist yet
            initializeQuestions();
            saveQuestionsToFile();
            return;
        }

        try (BufferedReader reader = new BufferedReader(new FileReader(file))) {
            String line;
            quizRepository.clear();
            while ((line = reader.readLine()) != null) {
                if (!line.trim().isEmpty()) {
                    Question q = Question.fromFileString(line);
                    if (q != null) quizRepository.add(q);
                }
            }
        } catch (IOException e) {
            System.out.println("Error reading questions file: " + e.getMessage());
        }
    }

    private static void saveQuestionsToFile() {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(QUESTIONS_FILE))) {
            for (Question q : quizRepository) {
                writer.write(q.toFileString());
                writer.newLine();
            }
        } catch (IOException e) {
            System.out.println("Error saving questions file: " + e.getMessage());
        }
    }

    private static void savePerformanceToFile(String studentName, int score, int total, double percentage) {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(PERFORMANCE_FILE, true))) {
            DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss");
            String timestamp = dtf.format(LocalDateTime.now());
            
            String logEntry = String.format("[%s] Student: %s | Score: %d/%d | Percentage: %.2f%%", 
                    timestamp, studentName, score, total, percentage);
            writer.write(logEntry);
            writer.newLine();
        } catch (IOException e) {
            System.out.println("Error updating performance logs: " + e.getMessage());
        }
    }

    private static void viewStudentPerformance() {
        File file = new File(PERFORMANCE_FILE);
        System.out.println("\n--- Historical Student Performance ---");
        if (!file.exists() || file.length() == 0) {
            System.out.println("No performance records found.");
            return;
        }

        try (BufferedReader reader = new BufferedReader(new FileReader(file))) {
            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            System.out.println("Error reading performance file: " + e.getMessage());
        }
    }

    // --- SEED DATA ---
    private static void initializeQuestions() {
        List<String> op1 = List.of("JDK", "JVM", "JRE", "JDB");
        quizRepository.add(new Question("Which component is responsible for running the compiled Java bytecode?", op1, 1));

        List<String> op2 = List.of("O(n log n)", "O(n)", "O(1)", "O(n^2)");
        quizRepository.add(new Question("What is the time complexity to access an element in an ArrayList by index?", op2, 2));
    }

    // --- TEACHER MODULE ---
    private static void teacherLogin() {
        System.out.println("\n--- Teacher Portal ---");
        System.out.print("Enter Username: ");
        String username = scanner.next();
        System.out.print("Enter Password: ");
        String password = scanner.next();
        scanner.nextLine(); // Clear buffer

        if (username.equals("teacher") && password.equals("admin123")) {
            System.out.println("Login Successful!");
            teacherMenu();
        } else {
            System.out.println("Invalid Credentials!");
        }
    }

    private static void teacherMenu() {
        while (true) {
            System.out.println("\n--- Teacher Dashboard ---");
            System.out.println("1. Add Question");
            System.out.println("2. View All Questions");
            System.out.println("3. Remove a Question");
            System.out.println("4. View Student Performance");
            System.out.println("5. Logout");
            System.out.print("Choose an option: ");
            
            int choice = getValidIntegerInput();
            scanner.nextLine(); // Clear buffer

            if (choice == 1) {
                System.out.print("Enter Question Text: ");
                String text = scanner.nextLine();
                
                List<String> options = new ArrayList<>();
                for (int i = 0; i < 4; i++) {
                    System.out.print("Enter Option " + (char)('A' + i) + ": ");
                    options.add(scanner.nextLine());
                }
                
                System.out.print("Enter Correct Option Index (1 for A, 2 for B, 3 for C, 4 for D): ");
                int correctIndex = getValidIntegerInput() - 1;

                if (correctIndex >= 0 && correctIndex < 4) {
                    quizRepository.add(new Question(text, options, correctIndex));
                    saveQuestionsToFile(); // Persist instantly
                    System.out.println("Question successfully updated into the quiz system!");
                } else {
                    System.out.println("Invalid correct index. Question not saved.");
                }
            } else if (choice == 2) {
                displayQuestions();
            } else if (choice == 3) {
                removeQuestion();
            } else if (choice == 4) {
                viewStudentPerformance();
            } else if (choice == 5) {
                System.out.println("Logged out from Teacher Portal.");
                break;
            } else {
                System.out.println("Invalid option.");
            }
        }
    }

    private static void displayQuestions() {
        if (quizRepository.isEmpty()) {
            System.out.println("No questions available.");
            return;
        }
        for (int i = 0; i < quizRepository.size(); i++) {
            Question q = quizRepository.get(i);
            System.out.println("\nQ" + (i + 1) + ": " + q.getQuestionText());
            char optChar = 'A';
            for (String opt : q.getOptions()) {
                System.out.println("  " + optChar + ") " + opt);
                optChar++;
            }
        }
    }

    private static void removeQuestion() {
        if (quizRepository.isEmpty()) {
            System.out.println("No questions available to remove.");
            return;
        }
        displayQuestions();
        System.out.print("\nEnter the question number you want to remove: ");
        int qNum = getValidIntegerInput();
        
        if (qNum > 0 && qNum <= quizRepository.size()) {
            quizRepository.remove(qNum - 1);
            saveQuestionsToFile(); // Persist structural update
            System.out.println("Question Q" + qNum + " was successfully removed!");
        } else {
            System.out.println("Invalid question number.");
        }
    }

    // --- STUDENT MODULE ---
    private static void studentLogin() {
        System.out.println("\n--- Student Portal ---");
        System.out.print("Enter Name: ");
        String name = scanner.next();
        
        System.out.println("Welcome, " + name + "! Get ready for the quiz.");
        takeQuiz(name);
    }

    private static void takeQuiz(String studentName) {
        if (quizRepository.isEmpty()) {
            System.out.println("No questions available to take right now. Ask your teacher!");
            return;
        }

        int score = 0;
        System.out.println("\n--- QUIZ START ---");

        for (int i = 0; i < quizRepository.size(); i++) {
            Question q = quizRepository.get(i);
            System.out.println("\nQ" + (i + 1) + ": " + q.getQuestionText());
            
            char optChar = 'A';
            for (String opt : q.getOptions()) {
                System.out.println("  " + optChar + ") " + opt);
                optChar++;
            }
            
            System.out.print("Your Answer (A/B/C/D): ");
            char answer = scanner.next().toUpperCase().charAt(0);
            int studentAnswerIndex = answer - 'A';

            if (studentAnswerIndex == q.getCorrectAnswerIndex()) {
                score++;
            }
        }

        // --- CALCULATE AND DISPLAY SCORE ---
        System.out.println("\n--- QUIZ RESULT ---");
        System.out.println("Student Name: " + studentName);
        System.out.println("Total Score: " + score + " / " + quizRepository.size());
        double percentage = ((double) score / quizRepository.size()) * 100;
        System.out.printf("Percentage: %.2f%%\n", percentage);

        // Track performance to file
        savePerformanceToFile(studentName, score, quizRepository.size(), percentage);
    }

    // Helper method to validate structural integrity of console inputs
    private static int getValidIntegerInput() {
        while (!scanner.hasNextInt()) {
            System.out.println("Invalid format! Please enter a number.");
            scanner.next();
        }
        return scanner.nextInt();
    }
}
