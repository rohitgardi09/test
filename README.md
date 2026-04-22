RELEASE NOTE
INTRODUCTION: -
This release introduces an enhancement in password generation logic for newly created users and encryption/API key files to improve overall security.
Overview: -
The existing password generation mechanism has been updated to enforce stricter security standards. The new logic ensures controlled character usage, mandatory inclusion of required character types, and improved randomness using secure generation techniques.
Code changes / Config changes: -
Updated Password Generation Logic:
Password length set to 8 characters
Allowed characters:
Uppercase: ABCDEFGHJKMNPQRSTUVWXYZ
Lowercase: abcdefghjkmnpqrstuvwxyz
Digits: 23456789
Special characters: . @ / , +
Rules implemented:
At least 1 uppercase, 1 lowercase, 1 digit, and 1 special character
Special character placed randomly (not in first position)
Remaining characters filled randomly from allowed set
Implemented using SecureRandom for better security
Code Update:
Modified PasswordGenerator.generatePassword() method as per new logic
Configuration Change:
Updated application.yml:
sbi.user.password.length changed from 12 to 8
Dependency Update:
commons_lang3 updated to 3.18.0
Previous Impact: -
Passwords were not following strict security rules
Password length was inconsistent (previously 12)
Character distribution was not controlled
Impact of Changes: -
Passwords now strictly follow 8-character secure format
Ensures mandatory inclusion of all character types
Improved randomness and security using SecureRandom
Meets enhanced security standards for user and API key generation
Rollback Plan: -
Revert changes in PasswordGenerator logic
Reset application.yml password length back to 12
Restore previous password generation behavior
