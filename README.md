Dear Sir/Madam,

I am currently implementing the Gen-7 changes in the DEV environment as per the migration guidelines.

I have the new EIS Gen-7 URL endpoint. However, I do not have the Client ID and Client Secret required to configure the request headers.

As mentioned, these three changes are required for Gen-7 migration:

1. New EIS Gen-7 URL endpoint
2. Client ID field in the request header
3. Client Secret field in the request header

Changes to be made:

- Update the API endpoint to the new EIS Gen-7 URL.
- Add the Client ID and Client Secret in the request headers.

Existing components to remain unchanged:

- Request body
- Existing business headers
- Encryption/decryption logic
- Backend processing
- Response structure

I have updated the Gen-7 URL endpoint, but the Client ID and Client Secret are not available with me yet. These credentials are required to proceed with the implementation and testing.

Thanks & Regards,
Rohit Gardi
