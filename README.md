Password generation logic for newly created user and encryption/API key file - Password should be 8 characters in length.

Allowed characters- Upper case letter ABCDEFGHJKMNPQRSTUVWXYZ

Lower case letter abcdefghjkmnpqrstuvwxyz

Special character .@/,+

Digits- 23456789

The password should contain 1 special character (randomly at any position except first position) and other characters randomly at random positions, should be in password at least once.

#### Implementation:

---

This ticket addresses the need to enhance the security of newly generated user passwords by implementing a more robust password generation logic. The current logic requires an update to meet the following security requirements:

**New Password Generation Requirements:**

* **Length:** Passwords must be exactly 8 characters in length.
* **Character Types:**
  * At least one uppercase character (ABCDEFGHJKMNPQRSTUVWXYZ).
  * At least one lowercase character (abcdefghjkmnpqrstuvwxyz).
  * At least one digit from 23456789.
  * One special character from .@/,+.
* **Special Character Placement:** The one special characters must be placed randomly within the password, with the exception that they cannot be in the first position.
* **Character Distribution:** All other characters (beyond the mandated uppercase, lowercase, digit, and special characters) should be randomly selected and placed at random positions within the password.
* **Minimum Inclusion:** Each required character type (uppercase, lowercase, digit, special character) must be present in the password at least once.

**Acceptance Criteria:**

* New user password generation adheres to all specified requirements.
* Automated tests are implemented to verify the new password generation logic.
* No regressions are introduced to existing user authentication or password management functionalities.

**Technical Notes:**

* Consider using a cryptographically secure random number generator for character selection and placement to ensure true randomness and prevent predictable patterns.
* Ensure that the special characters are selected from a predefined, secure set of allowed special characters.

##### Changes:

* Update com/epay/merchant/util/PasswordGenerator's generatePassword method logic as per above requirement
* Update application.yml config sbi.user.password.length value from 12 to 8 and same in all env.


# Code

src/main/java/com/epay/merchant/config/MerchantConfig.java

    @Value("${sbi.user.password.length:8}")
    private int passwordLength;
	
src/main/java/com/epay/merchant/util/PasswordGenerator.java

private static final SecureRandom random = new SecureRandom();
    private static final String DIGITS = "23456789";
    private static final String LOWER_CASE_CHARACTERS = "abcdefghjkmnpqrstuvwxyz";
    private static final String UPPER_CASE_CHARACTERS = "ABCDEFGHJKMNPQRSTUVWXYZ";
    private static final String SYMBOLS = ".@/,+";
    private static final String ALL = DIGITS + LOWER_CASE_CHARACTERS + UPPER_CASE_CHARACTERS + SYMBOLS;
    private static final char[] allArray = ALL.toCharArray();
    private static final char[] upperCase = UPPER_CASE_CHARACTERS.toCharArray();
    private static final char[] lowerCase = LOWER_CASE_CHARACTERS.toCharArray();
    private static final char[] digits = DIGITS.toCharArray();
    private static final char[] symbols = SYMBOLS.toCharArray();

    protected static final String DIGITS = "23456789";
    protected static final String LOWER_CASE_CHARACTERS = "abcdefghjkmnpqrstuvwxyz";
    protected static final String UPPER_CASE_CHARACTERS = "ABCDEFGHJKMNPQRSTUVWXYZ";
    protected static final String SYMBOLS = ".@/,+";
    protected static final String ALL = DIGITS + LOWER_CASE_CHARACTERS + UPPER_CASE_CHARACTERS ;
    protected static final char[] ALL_ARRAY = ALL.toCharArray();
    protected static final char[] UPPER_CASE_CHAR_ARRAY = UPPER_CASE_CHARACTERS.toCharArray();
    protected static final char[] LOWER_CASE_CHAR_ARRAY = LOWER_CASE_CHARACTERS.toCharArray();
    protected static final char[] NUMBER_ARRAY = DIGITS.toCharArray();
    protected static final char[] SYMBOLS_ARRAY = SYMBOLS.toCharArray();
	
	       sb.append(lowerCase[random.nextInt(lowerCase.length)]);
        sb.append(LOWER_CASE_CHAR_ARRAY[secure().randomInt(0, LOWER_CASE_CHAR_ARRAY.length)]);
        // get at least one uppercase letter
        sb.append(upperCase[random.nextInt(upperCase.length)]);
        sb.append(UPPER_CASE_CHAR_ARRAY[secure().randomInt(0, UPPER_CASE_CHAR_ARRAY.length)]);
        // get at least one digit
        sb.append(digits[random.nextInt(digits.length)]);
        // get at least one symbol
        sb.append(symbols[random.nextInt(symbols.length)]);
        // fill in remaining with random letters
        range(0, merchantConfig.getPasswordMaxLength() - 4).map(i -> allArray[random.nextInt(allArray.length)]).forEach(sb::append);
        return sb.toString();
        sb.append(NUMBER_ARRAY[secure().randomInt(0, NUMBER_ARRAY.length)]);

        //Generating remaining characters
        range(0, merchantConfig.getPasswordLength() - 4).mapToObj(i -> ALL_ARRAY[secure().randomInt(0, ALL_ARRAY.length - 1)]).forEach(sb::append);

        return sb.insert(secure().randomInt(1, 6), SYMBOLS_ARRAY[secure().randomInt(0, SYMBOLS_ARRAY.length)]).toString();
		
		
src/main/resources/application.yml

sbi.user.password.length: 12
sbi.user.password.length: 8
	
	
build.gradle

implementation "org.apache.commons:commons-lang3:${commons_lang3}"

gradle.properties
commons_lang3=3.18.0



mala hycha assa product note dee

   RELEASE NOTE   

 

INTRODUCTION: - 
   This release introduces mainly a unblock user and update profile changes as well as theme related changes and also include encryption key password length changes.


Overview: - 

This release focuses on enhancement to user management and customization, it introduce ability to 
Unblock users and include profile update functionality, in addition theme related changes have been implemented, release also include changes to encryption key password length

Code changes/config changes/DB changes: -  

    1. Implemented unlock user related Logic
    2. Added Update profile Functionality --  src/main/resources/application.yml - /user/unblock
    3. encryption key Password length changed to 8
    4. Modified theme related functionality logic – (get default theme if it doesn't exists)
    5. build.gradle -> addeded  testImplementation 'org.springframework.security:spring-security-test under dependancies
    6. gradle.properties ->  commons_lang3=3.18.0
    7. DB/Liquibase changes are there


Previous impact: -  
    1. Unable to unblock user 
    2. Unable to update profile
    3. Theme functionality having issue
    4. Encryption key generated OTP is showing 13 character (OTP should be 8 character)


Impact of changes: - 
    1. able to unblock user
    2. able to update profile update 
    3. theme functionality will work properly
    4. Encryption key will generated OTP is having 8 character


Rollback plan: -
    1. Revert service code changes.
    2. Restore previous behavior where unblock user and update profile remain unchanged.
