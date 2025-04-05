# W3C2PDF: A Specification for Bidirectional VC-PDF Conversion

**Authors**:  
- [Daev Mithran](https://github.com/DaevMithran)
    
## 1. Introduction

The adoption of W3C Verifiable Credentials (VCs) and Verifiable Presentations (VPs) is increasing, but human-readable formats require
specialized digital wallets for storage and verification. Many institutions and regulatory bodies require credentials in traditional document formats, such as PDF, for verification and archival purposes. This specification outlines bidirectional conversion methods between VCs/VPs and PDF, allowing both conversion from credentials to PDF documents and from PDF documents back to verifiable credentials, while preserving cryptographic verifiability and privacy characteristics in both directions.

## 2. Terminology

- **Verifiable Credential (VC)**: A cryptographically secure credential conforming to the W3C Verifiable Credentials Data Model.
- **Verifiable Presentation (VP)**: A privacy-preserving presentation derived from one or more Verifiable Credentials.
- **Proof**: Cryptographic material used to verify the authenticity and integrity of a credential.
- **QES**: Qualified Electronic Signature, as defined in eIDAS regulation.
- **DID**: Decentralized Identifier, a W3C standard for verifiable, decentralized digital identity.

## 3. Key Requirements

1. The PDF must contain a human-readable representation of the VC/VP
2. The VC data should be embedded inside the PDF metadata (XMP) or as an attachment (PDF/A-3) for machine readability
3. The PDF should support easy verification via QR codes, data URLs, or embedded proofs
4. The system must ensure data integrity, revocation handling, and interoperability with existing verification tools
5. The layout and design of the PDF should be dynamically determined based on credential format:
   - JSON-LD: Using context for dynamic UI generation
   - JWT: Using standardized templates
6. Allows verification of PDF credentials without requiring specialized wallet software

## 4. Technical Specification

This specification defines both conversion directions - from credentials to PDF and from PDF back to verifiable credentials.

![Workflow](https://raw.githubusercontent.com/DaevMithran/W3C2PDF/refs/heads/main/workflow.svg)

### 4.1 Credential to PDF Conversion

1. Extract VC/VP data from JSON-LD or JWT format
2. Determine the PDF layout:
   - For JSON-LD: Use context to apply appropriate rendering templates
   - For JWT: Apply standardized templates based on claim types
3. Render human-readable information using the selected template
4. Embed machine-readable VC/VP data:
   - Primary: Inside PDF XMP metadata
   - Secondary: As an attachment (for PDF/A-3 compliance)
5. Include verification components:
   - QR Code containing a verification URL, DID, or cryptographic hash
   - Machine-readable proof information
6. Preserve the original cryptographic proof from the VC/VP in the PDF metadata

### 4.2 Human-Readable Representation

#### 4.2.1 Default Rendering Approach

By default, both JWT and JSON-LD credentials will use standardized PDF rendering templates:

- Credential subject attributes shall be presented in a clear, organized layout
- Issuer information shall be prominently displayed
- Issuance and expiration dates shall be clearly indicated
- Credential type shall be identified
- Critical attributes must be visually emphasized
- Credential status must be clearly indicated if present

These default templates ensure consistent presentation across different types of credentials when no custom rendering is specified.

#### 4.2.2 JSON-LD Context-Driven Rendering (Optional)

JSON-LD credentials can leverage context URLs to select appropriate PDF templates:

1. **PDF Templates**
   - Real PDF files with professionally designed layouts and designated data placeholders
   - Hosted in the implementation's template library

2. **Context-Based Template Selection**
   - Implementation maps context URLs to specific template files
   - Example mapping: `"https://schema.org/ui-certificate" → certificate-template.pdf`
   - No additional properties needed in credential beyond standard context URLs

3. **Process**
   - System recognizes context URL in credential's `@context` array
   - Loads corresponding PDF template file
   - Maps credential data to template placeholders
   - Generates final PDF with combined template design and credential data

#### 4.2.3 Verifiable Presentation Rendering

Verifiable Presentations (VPs) often include multiple Verifiable Credentials (VCs) bundled under a single cryptographic proof. To balance cryptographic integrity with human readability, this specification recommends a **multi-page rendering format** with unified visual and verification elements.

- **Summary Page (First Page)**  
  A summary page lists all included credentials, displays the VP identifier, and includes a single verification mechanism (e.g., QR code or digital seal) that validates the entire presentation.

- **Credential Pages (Subsequent Pages)**  
  Each VC is rendered on its own page using a credential-specific template, preserving its unique visual identity and structure. Pages include consistent headers/footers with the VP identifier and pagination (e.g., “VP-12345 · Page 2 of 5”).

### 4.3 Machine-Readable Embedding

The PDF shall embed the original VC/VP data in three ways for maximum compatibility:

1. **XMP Metadata Embedding**

   The complete credential shall be stored in XMP metadata using a defined schema:

   ```xml
   <rdf:Description rdf:about=""
       xmlns:vc="http://www.w3.org/vc/1.0/">
       <vc:credential>BASE64_ENCODED_CREDENTIAL</vc:credential>
       <vc:format>JWT|JSON-LD</vc:format>
       <vc:version>1.0</vc:version>
       <vc:transformationMethod>DIRECT|DERIVED</vc:transformationMethod>
       <vc:transformationTimestamp>ISO8601_TIMESTAMP</vc:transformationTimestamp>
   </rdf:Description>
   ```

2. **File Attachment** (for PDF/A-3 compliance)

   The original credential is attached as an embedded file with appropriate MIME type:
   - `application/ld+json` for JSON-LD
   - `application/jwt` for JWT

3. **QR Code**

   A QR code containing either:
   - The complete credential (for smaller VCs)
   - A verification URL
   - A cryptographic hash linking to the credential

### 4.4 Cryptographic Proof Preservation

The PDF must preserve the original cryptographic proofs from the VC/VP:

1. **Original Proof Preservation**
   - For JWT: The complete token must be preserved intact within PDF metadata
   - For JSON-LD: The proof section must be preserved in its entirety
   - The original signature must remain verifiable after extraction from the PDF

2. **Additional Security Layers** (Implementation Choice)
   - Standard PDF digital signatures may be added as an additional layer if required
   - QES (Qualified Electronic Signature) for regulatory contexts requiring additional assurance

Implementations should prioritize preserving the integrity of the original credential proof. Additional security layers should not interfere with or invalidate the original proof verification.

### 4.5 PDF to Credential Conversion

#### 4.5.1 Credential Extraction

A PDF containing an embedded VC/VP shall support multiple extraction methods:

1. Primary extraction from XMP metadata
2. Secondary extraction from file attachment
3. Tertiary extraction from QR code (if present)

#### 4.5.2 Verification Workflow

Upon extraction:

1. The credential shall be verified against its cryptographic proof
2. The PDF signature shall be verified (QES and/or DID-based)
4. The extraction process shall confirm data integrity between visual and machine-readable components

#### 4.5.3 Verification Status

Verification shall result in one of these statuses:

- VERIFIED: Credential is cryptographically valid and PDF content matches
- TAMPERED: Visual content doesn't match embedded credential
- INVALID: Credential fails cryptographic verification
- EXPIRED: Credential is expired
- REVOKED: Credential has been revoked
- ERROR: Verification process couldn't complete

## 5. Implementation Guidelines

### 5.1 Implementation Requirements

Conforming implementations MUST:

- Preserve all cryptographically relevant material
- Maintain a one-to-one mapping between visible and embedded data
- Provide clear error handling for conversion failures

### 5.2 Security Considerations

This specification acknowledges these potential attack vectors:

- PDF manipulation to alter visible but not machine-readable content
- Metadata stripping attacks
- Social engineering using the PDF visual layer
- Attacks on the conversion process itself

Implementations must provide protections against these attacks.

## 6. Implementation Examples

### 6.1 Node.js Implementation Example

```javascript
async function generatePDF(credential, format) {
  const doc = new PDFDocument();
  doc.pipe(fs.createWriteStream('credential.pdf'));
  
  // Determine layout based on credential format
  ...
  
  // Add QR code for verification
  ...
  
  // Embed machine-readable credential
  ...
  
  // Attach the original credential as a file (for PDF/A-3)
  ...
  doc.end();
}

function generateXMPMetadata(credential, format) {
  ...
  return xmp.end({ pretty: true });
}

async function generateQRCode(credential) {
  // Create QR code with the credential or a verification URL
}
```

### 6.2 Verification Example

```javascript
async function verifyPDFCredential(pdfBuffer) {
    // Extract credential from PDF
    const credential = await extractCredentialFromPDF(pdfBuffer);
    const format = determineCredentialFormat(credential);
    
    // Verify the credential's cryptographic proof
    const isCredentialValid = await verifyCredential(credential, format);
    
    // Verify the PDF signature
    const isPDFSignatureValid = await verifyPDFSignature(pdfBuffer);
    
    // Check credential status (revocation)
    const isActive = await checkCredentialStatus(credential, format);
    
    // Verify data integrity between visual and machine-readable components
    const isDataIntegrityValid = await verifyDataIntegrity(pdfBuffer, credential);
    
    // Determine overall verification status
    ...
    
    return {
      status,
      details: {
          ...
      }
    };
}
```

## 7. Interoperability & Compliance

The specification ensures compatibility with:

- W3C Verifiable Credentials Data Model 1.1
- eIDAS 2.0 requirements for digital credentials
- ETSI standards for digital signatures
- PDF/A-3 for long-term archival
- Accessibility standards (PDF/UA) for universal access
- Common wallet implementations for credential import/export

## 8. Conclusion

This specification bridges the gap between self-sovereign identity and traditional document-based workflows through bidirectional conversion between verifiable credentials and PDF documents. By enabling both credential-to-PDF and PDF-to-credential transformations, it makes verifiable credentials more accessible and easy to verify without requiring specialized wallet software.

The bidirectional nature of this specification serves dual purposes:

1. **Credential to PDF**: Allows credentials to be shared in conventional document formats for archival and accessibility
2. **PDF to Credential**: Enables verification of these documents through conversion back to verifiable credentials, preserving the cryptographic security model

The specification supports a range of implementation approaches - from basic credential exchange scenarios to highly regulated environments requiring qualified signatures and compliance with specific trust frameworks. By supporting both JSON-LD and JWT formats with appropriate rendering approaches, it maximizes interoperability across the VC ecosystem while allowing implementations to select features appropriate to their specific requirements.

## 💖 Support This Project
Like this project? [Donate via PayPal](https://paypal.me/DaevMithran) and support continued improvements.