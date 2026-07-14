# Online-Quiz-System
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

// 1. Question Class to represent individual quiz questions
class Question {
    private String questionText;
    private List<String> options;
    private int correctAnswerIndex; // 0-based index (e.g., 0 for A, 1 for B)

    public Question(String questionText, List<String> options, int correctAnswerIndex) {
        this.questionText = questionText;
        this.options = options;
        this.correctAnswerIndex = correctAnswerIndex;
    }

    public String getQuestionText() { return questionText; }
    public List<String> getOptions() { return options; }
    public int getCorrectAnswerIndex() { return correctAnswerIndex; }
}

// 2. Main System Class
public class QuizSystem {
    private static List<Question> quizRepository = new ArrayList<>();
    private static Scanner scanner = new Scanner(System.in);

    public static void main(String[] args) {
        // Seed initial questions
        initializeQuestions();

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

    // --- SEED DATA ---
    private static void initializeQuestions() {
        List<String> op1 = List.of("JDK", "JVM", "JRE", "JDB");
        quizRepository.add(new Question("Which component is responsible for running the compiled Java bytecode?", op1, 1)); // JVM

        List<String> op2 = List.of("O(n log n)", "O(n)", "O(1)", "O(n^2)");
        quizRepository.add(new Question("What is the time complexity to access an element in an ArrayList by index?", op2, 2)); // O(1)
    }

    // --- TEACHER MODULE ---
    private static void teacherLogin() {
        System.out.println("\n--- Teacher Portal ---");
        System.out.print("Enter Username: ");
        String username = scanner.next();
        System.out.print("Enter Password: ");
        String password = scanner.next();
        scanner.nextLine(); // Clear buffer

        // Simple hardcoded credential check
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
            System.out.println("1. Add/Update Question");
            System.out.println("2. View All Questions");
            System.out.println("3. Logout");
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
                    System.out.println("Question successfully updated into the quiz system!");
                } else {
                    System.out.println("Invalid correct index. Question not saved.");
                }
            } else if (choice == 2) {
                displayQuestions();
            } else if (choice == 3) {
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

    // --- STUDENT MODULE ---
    private static void studentLogin() {
        System.out.println("\n--- Student Portal ---");
        System.out.print("Enter Name: ");
        String name = scanner.next(); // Simulating student identifier
        
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
