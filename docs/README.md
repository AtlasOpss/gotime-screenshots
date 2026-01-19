# Time Tracker Extension

Professional time tracking extension for Azure DevOps with advanced reporting, admin controls, and data management capabilities.

## Features

### 🔐 License Management
- RSA asymmetric encryption for secure licensing
- Support for multiple domains per license
- User count limitations and warnings
- Automatic license validation and expiration checks

### ⏱️ Time Tracking
- Calendar-based time entry interface
- Work item integration
- Period-based reporting
- Mathematical utilities for precise time calculations

### 📊 Reporting & Analytics
- Comprehensive time reports
- Export capabilities (PDF, Excel)
- Visual calendar interface
- Activity summaries

### 🛠️ Administration
- Collection-level settings storage
- User capacity management
- License activation/deactivation
- Settings navigation helpers

## Development

### Prerequisites
- Node.js 16+
- npm 8+
- Azure DevOps Extension SDK

### Installation
```bash
npm install
```

### Development Server
```bash
npm run start:dev
```

### Building
```bash
# Development build
npm run build:dev

# Production build
npm run build
```

### Testing
```bash
# Run all tests
npm test

# Run tests in watch mode
npm test:watch

# Run tests with coverage
npm test:coverage

# Run linting
npm run lint

# Fix linting issues
npm run lint:fix
```

### License Management Tools

#### Generate RSA Keys
```bash
npm run generate-keys
```

#### Generate Test Licenses
```bash
npm run generate-license
```

#### Generate License PDF
```bash
cd tools
node generate-licenses-pdf.js
```

## Project Structure

```
src/
├── components/           # React components
│   ├── effort-activity/  # Main calendar interface
│   ├── effort-report/    # Reporting interface
│   ├── effort-settings/  # Settings panel
│   ├── effort-dialog/    # Time entry dialog
│   └── common/          # Shared components
├── services/            # Business logic
│   ├── settings-storage.ts
│   ├── time-log-storage.ts
│   └── capacity-service.ts
├── utils/               # Utility functions
│   ├── license-utils.ts # License management
│   ├── sdk-helpers.ts   # Azure DevOps SDK helpers
│   └── math-utils.ts    # Mathematical calculations
└── __tests__/           # Unit tests
```

## License System

### Architecture
The extension uses RSA asymmetric encryption for secure license validation:

1. **Private Key**: Used by license generator to sign licenses
2. **Public Key**: Embedded in extension for signature verification
3. **License Format**: `base64_payload.signature.public_key`

### License Payload
```json
{
  "domains": ["devops.higgscloud.com", "localhost"],
  "userCount": 50,
  "expireDate": "2025-12-31",
  "issuedDate": "2025-09-11",
  "version": "1.0.0"
}
```

### Security Features
- Public key validation prevents tampering
- Constant-time string comparison prevents timing attacks
- Input sanitization and validation
- Domain case-insensitive matching
- Automatic expiration checks

## Testing

The project includes comprehensive unit tests covering:

### Math Utils
- Floating-point precision handling
- Decimal rounding functions
- Edge case validation

### Integration Tests
- Module import validation
- License format verification
- Base64 encoding/decoding
- URL parsing functionality

### Test Coverage
- **Lines**: Covered by Jest with HTML/Cobertura reports
- **Components**: Basic integration tests
- **Utils**: Comprehensive unit tests
- **Services**: Import validation

## Build & Deployment

### Webpack Configuration
- **Code Splitting**: Vendor chunks for optimized loading
- **Static Assets**: Automatic copying of HTML/CSS/images
- **Development**: Hot reloading with HTTPS support
- **Production**: Minification and optimization

### Extension Packaging
```bash
# Create VSIX package
npm run build

# Output: rdcdev.time-tracker-extension-{version}.vsix
```

### Version Management
Uses semantic versioning with Git tags:
```bash
git tag -a 0.2.0 -m "v0.2.0 - License system implementation"
git push origin 0.2.0
```

## CI/CD Pipeline

### GitLab CI/CD Stages

#### 1. Test Stage (Every Commit)
- **Triggers**: Push, MR, Tags
- **Actions**: 
  - Run unit tests (`npm test`)
  - Generate coverage reports
  - Upload JUnit and Cobertura reports
- **Artifacts**: Coverage reports (30 days)

#### 2. Build Stage (Tags Only)
- **Triggers**: Git tags only
- **Dependencies**: Test stage must pass
- **Actions**:
  - Production build (`npm run compile`)
  - Create VSIX package
  - Upload build artifacts

#### 3. Release Stage (Tags Only)
- **Triggers**: Git tags only
- **Dependencies**: Build stage must pass
- **Actions**:
  - Create GitLab release
  - Attach VSIX download link

### Pipeline Features
- **Test Coverage**: Automatic coverage reporting in MRs
- **JUnit Reports**: Test results integration
- **Artifact Storage**: 30-day retention for coverage reports
- **Cache Optimization**: `node_modules` caching per branch
- **Fail Fast**: Build only runs if tests pass

## Environment Variables

### License Generation (CI/CD)
- `PRIVATE_KEY`: RSA private key for license signing
- `PUBLIC_KEY`: RSA public key for verification

### Development
- `NODE_ENV`: Development environment flag
- `WEBPACK_SERVE`: Development server flag

## Contributing

### Code Style
- TypeScript strict mode
- ESLint configuration
- Prettier formatting
- Jest testing framework

### Git Workflow
1. Feature branches from `main`
2. Unit tests required
3. Code review process
4. Semantic versioning

### Security Guidelines
- No sensitive data in code
- License keys via environment variables
- Input validation for all user data
- Constant-time comparisons for crypto operations

## License

UNLICENSED - Private software for enterprise use.

## Support

For technical support and license requests, contact the RDC Development Team.
