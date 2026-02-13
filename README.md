# CodeStorm-SIBATHON-2026
VitalLink AI is a Java-based healthcare assistant that helps reduce hospital overcrowding by acting as a digital first responder. It evaluates user symptoms and classifies cases as Normal, Urgent, or Emergency, enabling doctors to prioritize critical patients and deliver faster, more efficient care.
import java.io.*;
import java.util.*;
import java.util.List;

/* =========================
   MODEL CLASSES
========================= */

class Doctor {
    String name;
    String specialization;
    int experience;
    String email;

    Doctor(String name, String specialization, int experience, String email) {
        this.name = name;
        this.specialization = specialization;
        this.experience = experience;
        this.email = email;
    }

    public String toString() {
        return name + " (" + specialization + ", " + experience + " yrs exp, Email: " + email + ")";
    }
}

class Disease {
    String name;
    String specialization;
    List<String> symptoms;

    Disease(String name, String specialization, List<String> symptoms) {
        this.name = name;
        this.specialization = specialization;
        this.symptoms = symptoms;
    }
}

class PatientExperience {
    String diseaseName;
    String severity;
    String notes;

    PatientExperience(String diseaseName, String severity, String notes) {
        this.diseaseName = diseaseName;
        this.severity = severity;
        this.notes = notes;
    }

    public String toFileString() {
        return diseaseName + "|" + severity + "|" + notes;
    }

    public static PatientExperience fromFileString(String line) {
        String[] parts = line.split("\\|");
        if (parts.length != 3) return null;
        return new PatientExperience(parts[0], parts[1], parts[2]);
    }
}

/* =========================
   MAIN CONSOLE APPLICATION
========================= */

public class docters {

    static Scanner sc = new Scanner(System.in);
    static final String HISTORY_FILE = "history.txt";

    static List<Disease> diseases = new ArrayList<>();
    static List<Doctor> doctors = new ArrayList<>();
    static List<PatientExperience> history = new ArrayList<>();

    public static void main(String[] args) {

        initializeData();
        loadHistory();

        System.out.println("=== Welcome to Console Telemedicine System ===");

        while (true) {
            System.out.println("\n1. Enter Symptoms");
            System.out.println("2. Show Patient History");
            System.out.println("3. Exit");
            System.out.print("Choice: ");

            int choice = Integer.parseInt(sc.nextLine());

            switch (choice) {
                case 1:
                    diagnosePatient();
                    break;
                case 2:
                    showHistory();
                    break;
                case 3:
                    saveHistory();
                    System.out.println("Thank you for using the system!");
                    System.exit(0);
                default:
                    System.out.println("Invalid choice!");
            }
        }
    }

    private static void initializeData() {
        diseases.add(new Disease("Dengue", "Infectious Disease",
                Arrays.asList("fever", "rash", "eye pain", "headache")));
        diseases.add(new Disease("Heart Disease", "Cardiologist",
                Arrays.asList("chest pain", "breathing problem", "fatigue")));
        diseases.add(new Disease("Skin Allergy", "Dermatologist",
                Arrays.asList("rash", "itching", "red skin")));

        doctors.add(new Doctor("Dr. Ahmed", "Infectious Disease", 10, "ahmed@example.com"));
        doctors.add(new Doctor("Dr. Sara", "Cardiologist", 8, "sara@example.com"));
        doctors.add(new Doctor("Dr. Ali", "Dermatologist", 6, "ali@example.com"));
    }

    private static void diagnosePatient() {
        System.out.print("Enter your symptoms (comma separated): ");
        String input = sc.nextLine().toLowerCase();
        String[] enteredSymptoms = input.split(",");

        Disease bestMatch = null;
        int maxMatch = 0;

        for (Disease disease : diseases) {
            int matchCount = 0;
            for (String s : enteredSymptoms) {
                if (disease.symptoms.contains(s.trim())) {
                    matchCount++;
                }
            }
            if (matchCount > maxMatch) {
                maxMatch = matchCount;
                bestMatch = disease;
            }
        }

        if (bestMatch == null) {
            System.out.println("No disease matched your symptoms.");
            return;
        }

        System.out.println("\nPossible Disease: " + bestMatch.name);
        System.out.println("Related Symptoms:");
        for (String s : bestMatch.symptoms) {
            System.out.println("- " + s);
        }

        String severity;
        if (maxMatch == 1) severity = "MILD";
        else if (maxMatch == 2) severity = "MODERATE";
        else severity = "SEVERE";

        System.out.println("Severity: " + severity);

        // Show doctors
        List<Doctor> relevantDoctors = new ArrayList<>();
        System.out.println("\nRecommended Doctors:");
        for (Doctor d : doctors) {
            if (d.specialization.equals(bestMatch.specialization)) {
                relevantDoctors.add(d);
                System.out.println(relevantDoctors.size() + ". " + d);
            }
        }

        if (!relevantDoctors.isEmpty()) {
            System.out.print("\nSelect a doctor to contact (number): ");
            int docChoice = Integer.parseInt(sc.nextLine());
            if (docChoice > 0 && docChoice <= relevantDoctors.size()) {
                Doctor selectedDoctor = relevantDoctors.get(docChoice - 1);
                System.out.println("\nYou can contact Dr. " + selectedDoctor.name + " at email: " + selectedDoctor.email);
                System.out.println("Copy-paste the following message in your email client:\n");
                System.out.println("Subject: Consultation Request");
                System.out.println("Body: Hello Dr. " + selectedDoctor.name + ", I am experiencing symptoms: " + input + ". Please advise. Thank you.");
            }
        }

        // Save patient experience in memory and file
        PatientExperience pe = new PatientExperience(bestMatch.name, severity, "Diagnosed via system");
        history.add(pe);
        appendHistoryToFile(pe);
    }

    private static void showHistory() {
        if (history.isEmpty()) {
            System.out.println("No patient history available.");
            return;
        }
        System.out.println("\n--- Patient History ---");
        int i = 1;
        for (PatientExperience pe : history) {
            System.out.println(i + ". Disease: " + pe.diseaseName + ", Severity: " + pe.severity + ", Notes: " + pe.notes);
            i++;
        }
    }

    private static void loadHistory() {
        File file = new File(HISTORY_FILE);
        if (!file.exists()) return;

        try (BufferedReader br = new BufferedReader(new FileReader(file))) {
            String line;
            while ((line = br.readLine()) != null) {
                PatientExperience pe = PatientExperience.fromFileString(line);
                if (pe != null) history.add(pe);
            }
        } catch (Exception e) {
            System.out.println("Failed to load history: " + e.getMessage());
        }
    }

    private static void saveHistory() {
        try (PrintWriter pw = new PrintWriter(new FileWriter(HISTORY_FILE))) {
            for (PatientExperience pe : history) {
                pw.println(pe.toFileString());
            }
        } catch (Exception e) {
            System.out.println("Failed to save history: " + e.getMessage());
        }
    }

    private static void appendHistoryToFile(PatientExperience pe) {
        try (PrintWriter pw = new PrintWriter(new FileWriter(HISTORY_FILE, true))) {
            pw.println(pe.toFileString());
        } catch (Exception e) {
            System.out.println("Failed to append history: " + e.getMessage());
        }
    }
}
