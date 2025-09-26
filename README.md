# Solana Wallet Vault API

A secure Node.js API that manages Solana wallets using HashiCorp Vault for secure storage. The API provides endpoints for wallet generation, public key retrieval, and transaction signing with both legacy and versioned transactions support.

## Features

- 🔐 Secure wallet storage in HashiCorp Vault
- 🔑 AppRole authentication for Vault access
- 🔒 JWT-based API authentication
- 📝 Support for both legacy and versioned Solana transactions
- 🛡️ AES-256-GCM encryption for public key transmission
- ⚡ Rate limiting and security headers
- 🚀 Express.js with ES Modules

## Prerequisites

- Node.js v20.x or later
- HashiCorp Vault server (OSS version)
- Solana cluster access (mainnet, testnet, or devnet)

## Installation

1. Clone the repository:
   ```bash
   git clone <repository-url>
   cd vault-solana-api
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Create a `.env` file in the root directory:
   ```env
   # Server Configuration
   PORT=3000
   NODE_ENV=development
   ALLOWED_ORIGINS=http://localhost:3000,https://yourdomain.com

   # JWT Configuration
   JWT_SECRET=your-jwt-secret-key-here

   # Vault Configuration
   VAULT_ADDR=http://your-vault-server:8200
   VAULT_ROLE_ID=your-role-id-here
   VAULT_SECRET_ID=your-secret-id-here

   # Encryption Configuration (32-byte hex key for AES-256-GCM)
   ENCRYPTION_KEY=your-32-byte-hex-encryption-key

   # Rate Limiting
   RATE_LIMIT_WINDOW_MS=900000
   RATE_LIMIT_MAX_REQUESTS=100
   ```

4. Start the server:
   ```bash
   npm start
   ```

## API Documentation

### Authentication

All API endpoints require a valid JWT token in the Authorization header:
```
Authorization: Bearer your-jwt-token
```

### Endpoints

#### 1. Generate Wallet
- **POST** `/api/wallet/generate`
- Creates a new Solana wallet and stores it in Vault
- **Request Body:**
  ```json
  {
    "userId": "user123"
  }
  ```
- **Response:**
  ```json
  {
    "userId": "user123",
    "encryptedPublicKey": "iv:encrypted:authTag"
  }
  ```

#### 2. Get Public Key
- **GET** `/api/wallet/public-key/:userId`
- Retrieves the encrypted public key for a user
- **Response:**
  ```json
  {
    "encryptedPublicKey": "iv:encrypted:authTag"
  }
  ```

#### 3. Sign Transaction
- **POST** `/api/wallet/sign-transaction`
- Signs a Solana transaction (supports both legacy and versioned transactions)
- **Request Body:**
  ```json
  {
    "userId": "user123",
    "encryptedPublicKey": "iv:encrypted:authTag",
    "serializedTransaction": "base64_encoded_transaction",
    "isVersioned": false
  }
  ```
- **Response:**
  ```json
  {
    "signedTransaction": "base64_encoded_signed_transaction"
  }
  ```

## Security Features

1. **Vault Security:**
   - AppRole authentication
   - Secure secret storage
   - Regular token rotation

2. **API Security:**
   - JWT authentication
   - Rate limiting
   - CORS protection
   - Helmet security headers
   - AES-256-GCM encryption for sensitive data

3. **Error Handling:**
   - Sanitized error messages in production
   - Detailed logging in development
   - Proper HTTP status codes

## Development

To run in development mode with auto-reload:
```bash
npm run dev
```

## HashiCorp Vault Setup

1. Enable KV v2 secret engine:
   ```bash
   vault secrets enable -version=2 -path=secret kv
   ```

2. Enable AppRole auth method:
   ```bash
   vault auth enable approle
   ```

3. Create a policy for wallet operations:
   ```hcl
   # wallet-policy.hcl
   path "secret/data/solana-wallets/*" {
     capabilities = ["create", "read", "update", "delete"]
   }
   ```
   ```bash
   vault policy write wallet-policy wallet-policy.hcl
   ```

4. Configure AppRole:
   ```bash
   vault write auth/approle/role/wallet-role \
       token_policies="wallet-policy" \
       token_ttl=1h \
       token_max_ttl=4h
   ```

5. Get Role ID and Secret ID:
   ```bash
   vault read auth/approle/role/wallet-role/role-id
   vault write -f auth/approle/role/wallet-role/secret-id
   ```

## Error Codes

- `400`: Bad Request - Missing or invalid parameters
- `401`: Unauthorized - Missing authentication token
- `403`: Forbidden - Invalid token or wallet access
- `404`: Not Found - Wallet or resource not found
- `500`: Internal Server Error - Server-side error

## Contributing

1. Fork the repository
2. Create your feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## License

This project is licensed under the MIT License. 
