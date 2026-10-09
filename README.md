@Bean
public LdapTemplate ldapTemplate(LdapContextSource contextSource) {
    LdapTemplate ldapTemplate = new LdapTemplate(contextSource);
    ldapTemplate.setIgnorePartialResultException(true);
    return ldapTemplate;
}
