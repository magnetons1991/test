import java.time.LocalDate;
import java.time.Period;
import java.util.*;
​
// ============================================================================
// ENUMS & MODELS
// ============================================================================
​
enum PEPClass {
    CLASS_1, CLASS_2, CLASS_3, CLASS_4, NON_PEP
}
​
enum MatchConclusion {
    MATCH_CONFIRMED("Conclusion: Match Confirmed"),
    PARTIAL_MATCH_ESCALATE("Conclusion: Partial Match / Requires Manual Escalation"),
    FALSE_POSITIVE("Conclusion: False Positive / No Match");
​
    private final String label;
    MatchConclusion(String label) { this.label = label; }
    public String getLabel() { return label; }
}
​
enum RiskAppetite {
    WITHIN_APPETITE("WITHIN APPETITE (Tolerable)"),
    OUTSIDE_APPETITE("OUTSIDE APPETITE (Intolerable)");
​
    private final String label;
    RiskAppetite(String label) { this.label = label; }
    public String getLabel() { return label; }
}
​
/**
 * KYC details of the target individual provided for screening.
 */
record TargetKYC(
    String fullName,
    LocalDate dateOfBirth,
    String nationality,
    String countryOfResidence
) {}
​
/**
 * Raw data extracted from automated system screening logs (Phase I & II input).
 */
class MatchLog {
    private String matchId;
    private String rawName;
    private LocalDate exactDob;
    private Integer estimatedBirthYear;
    private String location;
    private List<String> chargesList = new ArrayList<>();
    private List<String> rolesList = new ArrayList<>();
    private String professionalBackground;
    private String politicalAffiliation;
    private boolean isPep;
    private PEPClass pepClass;
    private String status;
    private LocalDate reviewDate;
    private String imageUrl;
    private List<String> sourceLinks = new ArrayList<>();
​
    public MatchLog(String matchId, String rawName) {
        this.matchId = matchId;
        this.rawName = rawName;
    }
​
    // Getters and Setters
    public String getMatchId() { return matchId; }
    public String getRawName() { return rawName; }
    public LocalDate getExactDob() { return exactDob; }
    public void setExactDob(LocalDate exactDob) { this.exactDob = exactDob; }
    public Integer getEstimatedBirthYear() { return estimatedBirthYear; }
    public void setEstimatedBirthYear(Integer estimatedBirthYear) { this.estimatedBirthYear = estimatedBirthYear; }
    public String getLocation() { return location; }
    public void setLocation(String location) { this.location = location; }
    public List<String> getChargesList() { return chargesList; }
    public List<String> getRolesList() { return rolesList; }
    public String getProfessionalBackground() { return professionalBackground; }
    public void setProfessionalBackground(String professionalBackground) { this.professionalBackground = professionalBackground; }
    public String getPoliticalAffiliation() { return politicalAffiliation; }
    public void setPoliticalAffiliation(String politicalAffiliation) { this.politicalAffiliation = politicalAffiliation; }
    public boolean isPep() { return isPep; }
    public void setPep(boolean pep) { isPep = pep; }
    public PEPClass getPepClass() { return pepClass; }
    public void setPepClass(PEPClass pepClass) { this.pepClass = pepClass; }
    public String getStatus() { return status; }
    public void setStatus(String status) { this.status = status; }
    public LocalDate getReviewDate() { return reviewDate; }
    public void setReviewDate(LocalDate reviewDate) { this.reviewDate = reviewDate; }
    public String getImageUrl() { return imageUrl; }
    public void setImageUrl(String imageUrl) { this.imageUrl = imageUrl; }
    public List<String> getSourceLinks() { return sourceLinks; }
}
​
/**
 * Phase III: Independent 12-Point Profile structure.
 */
class MatchProfile {
    private String matchId;
    private String fullName;                   // 1
    private String dob;                        // 2
    private String estimatedDob;               // 3
    private String location;                   // 4
    private String accusations;                // 5
    private String additionalRoles;            // 6
    private String professionalBackground;     // 7
    private String politicalAffiliation;       // 8
    private String pepClassification;          // 9
    private String statusAndReviewDate;        // 10
    private String image;                      // 11
    private String sourcesFormatted;           // 12
    private String summaryParagraph;
​
    public MatchProfile(MatchLog log) {
        this.matchId = log.getMatchId();
        this.fullName = log.getRawName() != null ? log.getRawName() : "Not found";
        this.dob = log.getExactDob() != null ? log.getExactDob().toString() : "Not found";
        this.estimatedDob = log.getEstimatedBirthYear() != null ? log.getEstimatedBirthYear().toString() : "Not found";
        this.location = log.getLocation() != null ? log.getLocation() : "Not found";
        
        this.accusations = log.getChargesList().isEmpty() 
            ? "None found" 
            : String.join("; ", log.getChargesList());
            
        this.additionalRoles = log.getRolesList().isEmpty() 
            ? "None found" 
            : String.join("; ", log.getRolesList());
            
        this.professionalBackground = log.getProfessionalBackground() != null ? log.getProfessionalBackground() : "Not found";
        this.politicalAffiliation = log.getPoliticalAffiliation() != null ? log.getPoliticalAffiliation() : "Not found";
        
        this.pepClassification = log.isPep() 
            ? "PEP Identified (" + (log.getPepClass() != null ? log.getPepClass().name() : "Unclassified") + ")" 
            : "Non-PEP";
            
        this.statusAndReviewDate = "Status: " + (log.getStatus() != null ? log.getStatus() : "N/A") 
            + " | Review Date: " + (log.getReviewDate() != null ? log.getReviewDate().toString() : "N/A");
            
        this.image = log.getImageUrl() != null ? log.getImageUrl() : "No image found";
        
        this.sourcesFormatted = log.getSourceLinks().isEmpty() 
            ? "Sources - None" 
            : "Sources - " + String.join(", ", log.getSourceLinks());
            
        this.summaryParagraph = generateSummary(log);
    }
​
    private String generateSummary(MatchLog log) {
        if (log.getProfessionalBackground() == null && log.getChargesList().isEmpty() && log.getImageUrl() == null) {
            return "No personal or professional information is available, no images found.";
        }
        return String.format("Profile for %s associated with %s location. Charges recorded: %d. PEP Status: %s.",
                fullName, location, log.getChargesList().size(), pepClassification);
    }
​
    public String getMatchId() { return matchId; }
​
    @Override
    public String toString() {
        return String.format("""
            === 12-POINT PROFILE [Match ID: %s] ===
            1. Full Name: %s
            2. Date of Birth (DOB): %s
            3. Estimated DOB: %s
            4. Location: %s
            5. Accusations/Investigations: %s
            6. Additional Roles/Functions: %s
            7. Professional Background: %s
            8. Political Affiliation: %s
            9. PEP Classification: %s
            10. Status and Review Date: %s
            11. Image: %s
            12. %s
            
            Summary: %s
            """, matchId, fullName, dob, estimatedDob, location, accusations, 
                 additionalRoles, professionalBackground, politicalAffiliation, 
                 pepClassification, statusAndReviewDate, image, sourcesFormatted, summaryParagraph);
    }
}
​
// ============================================================================
// CORE AML PROCESSING ENGINE
// ============================================================================
​
public class AmlComplianceEngine {
​
    // Define intolerable risk keywords based on Risk Appetite Classification Matrix
    private static final Set<String> INTOLERABLE_OFFENSES = Set.of(
        "money laundering", "terrorism financing", "narcotic", "child pornography",
        "embezzlement", "fraud", "wire fraud", "bribery", "corruption", "cybercrime",
        "murder", "manslaughter", "ponzi", "pyramid scheme", "securities fraud",
        "forgery", "sex offender", "domestic violence", "burglary", "arson", "robbery"
    );
​
    /**
     * Executes the complete AML validation workflow across provided Match IDs.
     */
    public void processScreening(TargetKYC target, List<MatchLog> matchLogs) {
        System.out.println("================================================================================");
        System.out.println("                  PHASE III: INDEPENDENT PROFILING                              ");
        System.out.println("================================================================================");
​
        List<MatchProfile> profiles = new ArrayList<>();
        
        // HARD ISOLATION RULE: Treat every Match ID as a unique independent individual
        for (MatchLog log : matchLogs) {
            MatchProfile profile = new MatchProfile(log);
            profiles.add(profile);
            System.out.println(profile);
        }
​
        System.out.println("================================================================================");
        System.out.println("## AML Identity Comparison & False Positive Analysis");
        System.out.println("================================================================================");
​
        for (MatchLog log : matchLogs) {
            evaluateMatch(target, log);
        }
    }
​
    /**
     * Executes Phase IV Comparison and Risk Appetite Routing.
     */
    private void evaluateMatch(TargetKYC target, MatchLog log) {
        System.out.println("\n### Match ID: " + log.getMatchId());
​
        // Step 1: Parameters Evaluation
        String nameMatch = evaluateNameMatch(target.fullName(), log.getRawName());
        boolean chronoFeasible = evaluateChronology(target, log);
        boolean geoAligned = evaluateGeography(target, log);
​
        System.out.println("- Name Match Type: " + nameMatch);
        System.out.println("- Chronological Feasibility: " + (chronoFeasible ? "Feasible" : "Infeasible / Age Discrepancy"));
        System.out.println("- Geographic & National Alignment: " + (geoAligned ? "Aligned" : "No geographic alignment"));
​
        // Step 2: Match Conclusion Declaration
        MatchConclusion conclusion;
        if (nameMatch.equalsIgnoreCase("Exact Script Match") && chronoFeasible && geoAligned) {
            conclusion = MatchConclusion.MATCH_CONFIRMED;
        } else if (nameMatch.contains("Phonetic") || (chronoFeasible && !geoAligned)) {
            conclusion = MatchConclusion.PARTIAL_MATCH_ESCALATE;
        } else {
            conclusion = MatchConclusion.FALSE_POSITIVE;
        }
​
        System.out.println("\n" + conclusion.getLabel());
​
        // Step 3: Conditional Risk Appetite Assessment Routing
        if (conclusion == MatchConclusion.FALSE_POSITIVE || conclusion == MatchConclusion.PARTIAL_MATCH_ESCALATE) {
            System.out.println("Risk Appetite Assessment skipped: Subject identified as a " 
                + (conclusion == MatchConclusion.FALSE_POSITIVE ? "False Positive / Unconfirmed Match." : "Partial Match requiring manual escalation."));
        } else {
            // Apply Risk Matrix strictly on Confirmed Matches
            assessRiskAppetite(log);
        }
    }
​
    private String evaluateNameMatch(String targetName, String matchName) {
        if (targetName == null || matchName == null) return "No Match";
        if (targetName.equalsIgnoreCase(matchName)) {
            return "Exact Script Match";
        } else if (targetName.substring(0, 3).equalsIgnoreCase(matchName.substring(0, Math.min(3, matchName.length())))) {
            return "Phonetic / Partial Name Match";
        }
        return "Mismatch";
    }
​
    private boolean evaluateChronology(TargetKYC target, MatchLog log) {
        if (target.dateOfBirth() == null) return true;
        
        if (log.getExactDob() != null) {
            return target.dateOfBirth().equals(log.getExactDob());
        }
        
        if (log.getEstimatedBirthYear() != null) {
            int targetYear = target.dateOfBirth().getYear();
            return Math.abs(targetYear - log.getEstimatedBirthYear()) <= 3;
        }
        
        return true; // Feasible by default if DOB is missing from logs
    }
​
    private boolean evaluateGeography(TargetKYC target, MatchLog log) {
        if (log.getLocation() == null) return false;
        String locUpper = log.getLocation().toUpperCase();
        return locUpper.contains(target.nationality().toUpperCase()) || 
               locUpper.contains(target.countryOfResidence().toUpperCase());
    }
​
    private void assessRiskAppetite(MatchLog log) {
        RiskAppetite riskStatus = RiskAppetite.WITHIN_APPETITE;
        String primaryRiskFactor = "Low Risk / Tolerable Offenses";
​
        for (String charge : log.getChargesList()) {
            String lowerCharge = charge.toLowerCase();
            for (String intolerableKeyword : INTOLERABLE_OFFENSES) {
                if (lowerCharge.contains(intolerableKeyword)) {
                    riskStatus = RiskAppetite.OUTSIDE_APPETITE;
                    primaryRiskFactor = "Offense identified outside risk appetite: " + charge;
                    break;
                }
            }
        }
​
        System.out.println("Risk Assessment Declaration:");
        System.out.println("- Risk Level: " + riskStatus.getLabel());
        System.out.println("- Details: " + primaryRiskFactor);
    }
​
    // ============================================================================
    // MAIN DEMONSTRATION RUNNER
    // ============================================================================
​
    public static void main(String[] args) {
        // Sample Target KYC
        TargetKYC target = new TargetKYC(
            "John Doe", 
            LocalDate.of(1985, 6, 15), 
            "United States", 
            "United States"
        );
​
        // Sample Match Logs from Automated System
        MatchLog match1 = new MatchLog("MATCH-001", "John Doe");
        match1.setExactDob(LocalDate.of(1985, 6, 15));
        match1.setLocation("United States, New York");
        match1.getChargesList().add("Wire Fraud and Money Laundering");
        match1.setProfessionalBackground("Former Corporate Executive");
        match1.setStatus("ACTIVE");
        match1.setReviewDate(LocalDate.now());
        match1.getSourceLinks().add("https://sanctions.example.gov/ref/001");
​
        MatchLog match2 = new MatchLog("MATCH-002", "Jonathon Doe");
        match2.setEstimatedBirthYear(1955);
        match2.setLocation("United Kingdom, London");
        match2.setPep(true);
        match2.setPepClass(PEPClass.CLASS_1);
        match2.setProfessionalBackground("Senior Military Officer");
        match2.setStatus("MONITORED");
        match2.setReviewDate(LocalDate.now());
​
        List<MatchLog> screeningLogs = List.of(match1, match2);
​
        // Execute Analysis
        AmlComplianceEngine engine = new AmlComplianceEngine();
        engine.processScreening(target, screeningLogs);
