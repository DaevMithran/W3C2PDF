# W3C2PDF: A Specification for Bidirectional VC-PDF Conversion

**Authors**:  
- [Daev Mithran](https://github.com/DaevMithran)
    
## 1. Introduction

### 1.1 Motivation

The adoption of W3C Verifiable Credentials (VCs) and Verifiable Presentations (VPs) is increasing, but human-readable formats remain a barrier to accessibility. Many institutions and regulatory bodies require credentials in traditional document formats, such as PDF, for verification and archival purposes. This specification outlines bidirectional conversion methods between VCs/VPs and PDF - allowing both conversion from credentials to PDF documents and from PDF documents back to verifiable credentials, while preserving cryptographic verifiability and privacy characteristics in both directions.

### 1.2 Scope & Assumptions

- Supports bidirectional conversion between credentials and PDFs (VC/VP ↔ PDF)
- Supports both JSON-LD and JWT-based VCs & VPs for maximum interoperability
- The PDF contains both human-readable and machine-readable representations of the credential
- Supports all parties in the VC ecosystem: issuers, holders, and verifiers
- Compatible with various standards including PDF/A archival standards
- The UI of the PDF can be dynamically determined based on the JSON-LD context, allowing customized rendering per credential type
- JWT-based credentials follow standardized templates for consistent visual representation
- Supports various signature methods including standard digital signatures, QES for regulatory contexts, and DID-based approaches
- Allows verification of PDF credentials without requiring specialized wallet software

## 2. Terminology

- **Verifiable Credential (VC)**: A cryptographically secure credential conforming to the W3C Verifiable Credentials Data Model.
- **Verifiable Presentation (VP)**: A privacy-preserving presentation derived from one or more Verifiable Credentials.
- **PDF Document**: A Portable Document Format file conforming to ISO 32000-2:2020.
- **Conversion**: The process of transforming a VC/VP to a PDF document or vice versa.
- **Proof**: Cryptographic material used to verify the authenticity and integrity of a credential.
- **Selective Disclosure**: The ability to reveal only a subset of credential attributes.
- **QES**: Qualified Electronic Signature, as defined in eIDAS regulation.
- **DID**: Decentralized Identifier, a W3C standard for verifiable, decentralized digital identity.

## 3. Key Requirements

### 3.1 Bidirectional Conversion Requirements

1. The PDF must contain a human-readable representation of the VC/VP
2. The VC data should be embedded inside the PDF metadata (XMP) or as an attachment (PDF/A-3)
3. The PDF must be digitally signed using QES for legal recognition when required
4. The PDF should support easy verification via QR codes, data URLs, or embedded proofs
5. The system must ensure data integrity, revocation handling, and interoperability with existing verification tools
6. The layout and design of the PDF should be dynamically determined based on credential format:
   - JSON-LD: Using context for dynamic UI generation
   - JWT: Using standardized templates

## 4. Technical Specification

This specification defines both conversion directions - from credentials to PDF and from PDF back to verifiable credentials.

### 4.1 Credential to PDF Conversion

#### 4.1.1 General Structure

A PDF document derived from a VC/VP shall contain:

1. A human-readable representation of the credential data
2. Machine-readable metadata embedded in the PDF
3. Cryptographic proof information
4. Visual security elements

The rendering approach differs based on credential format:
- JSON-LD credentials shall use their context definitions to dynamically determine rendering
- JWT credentials shall follow a standardized template approach

#### 4.1.2 Generation Process

1. Extract VC/VP data from JSON-LD or JWT format
2. Determine the PDF layout:
   - For JSON-LD: Use context to apply appropriate rendering templates
   - For JWT: Apply standardized templates based on claim types
3. Render human-readable information using the selected template
4. Embed machine-readable VC/VP data:
   - Primary: Inside PDF XMP metadata
   - Secondary: As an attachment (for PDF/A-3 compliance)
   - Optional: As an invisible text layer for redundancy
5. Include verification components:
   - QR Code containing a verification URL, DID, or cryptographic hash
   - Machine-readable proof information
   - Visual verification instructions
6. Preserve the original cryptographic proof from the VC/VP in the PDF metadata

### 4.2 Human-Readable Representation

#### 4.2.1 JSON-LD Dynamic Rendering

For JSON-LD credentials, the PDF rendering shall be determined dynamically using:

- The `@context` definitions to interpret semantic meaning of properties
- Type definitions to determine appropriate visual layouts
- Schema.org or other vocabulary mappings to guide display formatting
- Optional display mapping extensions in the context

The context may contain rendering hints such as:
```json
{
  "@context": {
    "@version": 1.1,
    "ex": "https://example.org/",
    "name": "ex:name",
    "display": {
      "@id": "ex:display",
      "@context": {
        "order": "ex:displayOrder",
        "section": "ex:displaySection",
        "format": "ex:displayFormat"
      }
    },
    "name": {
      "@id": "ex:name",
      "display": {
        "order": 1,
        "section": "header",
        "format": "heading"
      }
    }
  }
}
```

#### 4.2.2 JWT Standardized Rendering

For JWT credentials, the PDF shall present credential data in a structured, standardized format:

- Credential subject attributes shall be presented in a clear, organized layout
- Issuer information shall be prominently displayed
- Issuance and expiration dates shall be clearly indicated
- Credential type shall be identified

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

### 4.6 Selective Disclosure Handling

When converting a VP with selective disclosure to PDF:

1. Only the disclosed attributes shall appear in the human-readable representation
2. The selective disclosure proofs shall be preserved in the embedded VP
3. The PDF shall indicate that it represents a partial disclosure of the credential
4. When extracting a VP from a PDF, the selective disclosure properties shall remain intact
5. For ZKP-based selective disclosure (e.g., BBS+ signatures):
   - The derived proof shall be preserved
   - The conversion process shall not compromise zero-knowledge properties

## 5. Implementation Guidelines

### 5.1 Context Extension for Display

This specification defines a display extension for JSON-LD contexts:

```json
{
  "@context": {
    "display": "https://w3id.org/vc-display/v1",
    "display:layout": {
      "@id": "https://w3id.org/vc-display/v1#layout",
      "@context": {
        "template": "https://w3id.org/vc-display/v1#template",
        "card": "https://w3id.org/vc-display/v1#card",
        "certificate": "https://w3id.org/vc-display/v1#certificate"
      }
    },
    "display:property": {
      "@id": "https://w3id.org/vc-display/v1#property",
      "@context": {
        "order": "https://w3id.org/vc-display/v1#order",
        "section": "https://w3id.org/vc-display/v1#section",
        "emphasize": "https://w3id.org/vc-display/v1#emphasize",
        "label": "https://w3id.org/vc-display/v1#label",
        "value": "https://w3id.org/vc-display/v1#value"
      }
    },
    "display:branding": {
      "@id": "https://w3id.org/vc-display/v1#branding",
      "@context": {
        "logo": "https://w3id.org/vc-display/v1#logo",
        "background": "https://w3id.org/vc-display/v1#background",
        "primaryColor": "https://w3id.org/vc-display/v1#primaryColor",
        "secondaryColor": "https://w3id.org/vc-display/v1#secondaryColor"
      }
    }
  }
}
```

### 7.2 Implementation Requirements

Conforming implementations MUST:

- Preserve all cryptographically relevant material
- Maintain a one-to-one mapping between visible and embedded data
- Provide clear error handling for conversion failures
- Implement at least one digital signature method for PDF integrity
- Support at least one of:
  - JSON-LD credentials with context-based dynamic rendering
  - JWT credentials with standardized template rendering
- Additional security measures (PDF signatures, QES) based on regulatory requirements- PDF/A-3 for long-term archival needs

### 5.3 Security Considerations

This specification acknowledges these potential attack vectors:

- PDF manipulation to alter visible but not machine-readable content
- Metadata stripping attacks
- Social engineering using the PDF visual layer
- Attacks on the conversion process itself

Implementations must provide protections against these attacks.

### 5.4 Privacy Considerations

- Implementers shall ensure no additional PII is leaked during conversion
- Metadata shall be protected against unintentional exposure
- Selective disclosure properties shall be preserved
- PDF properties shall be sanitized to remove identifying information not present in the original VC

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
    let status = 'ERROR';
    if (isCredentialValid && isPDFSignatureValid && isActive && isDataIntegrityValid) {
      status = 'VERIFIED';
    } else if (!isCredentialValid) {
      status = 'INVALID';
    } else if (!isPDFSignatureValid || !isDataIntegrityValid) {
      status = 'TAMPERED';
    } else if (!isActive) {
      status = 'REVOKED';
    }
    
    return {
      status,
      details: {
          ...
      }
    };
}

// Functions for extraction, verification, etc.
```

## 7. Interoperability & Compliance

The specification ensures compatibility with:

- W3C Verifiable Credentials Data Model 1.1
- eIDAS 2.0 requirements for digital credentials
- ETSI standards for digital signatures
- PDF/A-3 for long-term archival
- Accessibility standards (PDF/UA) for universal access
- Common wallet implementations for credential import/export

## 8. Future Considerations

This specification acknowledges these areas for future development:

- Advanced rendering capabilities for complex credential types
- Enhanced selective disclosure mechanisms
- Integration with emerging trust frameworks
- Support for credential schemas and validation
- Credential chaining and derived credentials
- Long-term archival considerations
- Internationalization and localization support
- Accessibility enhancements
- Backup and recovery mechanisms

## 9. Conclusion

This specification bridges the gap between self-sovereign identity and traditional document-based workflows through bidirectional conversion between verifiable credentials and PDF documents. By enabling both credential-to-PDF and PDF-to-credential transformations, it makes verifiable credentials more accessible and easy to verify without requiring specialized wallet software.

The bidirectional nature of this specification serves dual purposes:

1. **Credential to PDF**: Allows credentials to be shared in conventional document formats for archival and accessibility
2. **PDF to Credential**: Enables verification of these documents through conversion back to verifiable credentials, preserving the cryptographic security model

The specification supports a range of implementation approaches - from basic credential exchange scenarios to highly regulated environments requiring qualified signatures and compliance with specific trust frameworks. By supporting both JSON-LD and JWT formats with appropriate rendering approaches, it maximizes interoperability across the VC ecosystem while allowing implementations to select features appropriate to their specific requirements.
