# Codebase Walkthrough: Supply Chain Blockchain Project

## Project Structure

```
Supply-Chain-Blockchain/
├── src/
│   ├── Smart-Contract/          # Blockchain layer
│   │   ├── Contracts/           # Solidity smart contracts
│   │   ├── migrations/          # Deployment scripts
│   │   ├── test/                # Smart contract tests
│   │   ├── ABI/                 # Compiled contract ABIs
│   │   └── truffle-config.js    # Truffle configuration
│   │
│   ├── Components/              # React components
│   │   ├── Cards/               # Display components
│   │   ├── Modals/              # Dialog components
│   │   └── Toast/               # Notification components
│   │
│   ├── Pages/                   # React pages
│   │   ├── Admin/               # Admin verification
│   │   ├── Dashboard/           # Main dashboard
│   │   ├── Farmers/             # Farmer management
│   │   ├── Manufacturers/       # Manufacturer management
│   │   ├── Products/            # Product tracking
│   │   ├── Profile.js           # User profile
│   │   └── Register.js          # Registration
│   │
│   ├── Services/                # Business logic
│   │   ├── Contexts/            # React contexts (Auth, Contract)
│   │   ├── Actions/             # Redux actions
│   │   ├── Reducers/            # Redux reducers
│   │   └── Utils/               # Utility functions
│   │
│   ├── Layouts/                 # Layout components
│   ├── Assests/                 # Images, styles
│   └── App.js                   # Main app component
│
├── public/                      # Static files
├── package.json                 # Dependencies
└── README.md                    # Documentation
```

---

## Smart Contracts Layer

### **1. Admin.sol** (Base Contract)
**Purpose**: Manages admin privileges

```solidity
contract Admin {
    address public admin;
    
    modifier onlyAdmin {
        require(msg.sender == admin);
        _;
    }
}
```

**Key Functions**:
- `isAdmin()`: Check if address is admin
- `onlyAdmin` modifier: Restrict functions to admin

---

### **2. Stakeholder.sol** (Access Control)
**Purpose**: Base class for all supply chain participants

**Key Structures**:
```solidity
struct stakeholder {
    address id;           // Ethereum address
    string name;          // Name
    string location;      // Location
    string role;          // Role (Farmer, Manufacturer, etc.)
    bool isVerified;      // Verified by admin
}
```

**Key Functions**:
- `register()`: Register new stakeholder
- `verify()`: Admin verifies stakeholder
- `transferFrom()`: Transfer product ownership
- `setApprovalForAll()`: Approve operator for transfers
- `isApprovedForAll()`: Check if operator approved

**Access Control Modifiers**:
- `onlyStakeholder()`: Only registered stakeholders
- `onlyOwnerOrApproved()`: Only owner or approved operator

---

### **3. Farmer.sol** (Producer)
**Purpose**: Manages farmers and raw products

**Key Structures**:
```solidity
mapping(address => string[]) _farmerRawProducts;  // Products farmer produces
mapping(string => address[]) _rawProductFarmers;  // Farmers producing product
```

**Key Functions**:
- `registerFarmer()`: Register farmer with raw products
- `addRawProduct()`: Add new raw product
- `getFarmer()`: Get farmer details
- `getRawProductFarmers()`: Get farmers producing specific product

**Example**: Farmer registers "Milk" and "Cocoa" as raw products

---

### **4. Manufacturer.sol** (Processor)
**Purpose**: Manages manufacturers and product processing

**Key Structures**:
```solidity
struct rawProduct {
    string name;
    address[] boughtFromIds;  // Suppliers
    bool isVerified;          // All suppliers verified
}

mapping(address => uint256[]) _launchedProducts;  // Products launched
```

**Key Functions**:
- `register()`: Register manufacturer
- `addRawProduct()`: Add raw material with suppliers
- `launchProduct()`: Launch finished product
- `updateEnergy()`: Mark if using renewable energy

**Example**: Manufacturer receives verified Milk from Farmer, creates "Dairy Milk"

---

### **5. Product.sol** (Product Lifecycle)
**Purpose**: Manages products, transactions, and reviews

**Key Structures**:
```solidity
struct Item {
    uint256 id;              // Product ID
    string title;            // Product name
    address manufacturer;    // Creator
    address currentOwner;    // Current owner
    address lastOwner;       // Previous owner
    uint256 rating;          // Average rating
    uint launchDate;         // Creation date
    string image_url;        // Product image
}

struct Transaction {
    uint256 txId;
    address from;
    address to;
    uint date;
}

struct Review {
    uint256 id;
    uint date;
    uint256 rating;
    string comment;
    address reviewer;
}
```

**Key Functions**:
- `add()`: Create new product
- `transferProduct()`: Transfer ownership
- `addReview()`: Add immutable review
- `getProductHistory()`: Get all transactions
- `getProductReviews()`: Get all reviews

---

### **6. Main.sol** (Orchestrator)
**Purpose**: Connects all contracts

```solidity
contract Main {
    Farmer public farmer;
    Manufacturer public manufacturer;
    Stakeholder public stakeholder;
}
```

---

## Frontend Layer (React)

### **Authentication Flow**
```
App.js
  ↓
AuthContextProvider (Manages user login/logout)
  ↓
ContractContextProvider (Manages blockchain connection)
  ↓
Pages (Dashboard, Farmers, Manufacturers, Products, etc.)
```

### **Key Services**

#### **AuthContext.js**
- Manages user authentication
- Stores user role (Farmer, Manufacturer, Admin, Consumer)
- Handles MetaMask wallet connection

#### **ContractContext.js**
- Manages Web3 connection
- Loads smart contract ABIs
- Provides contract methods to components

#### **Utils**
- `stakeholder.js`: Stakeholder-related functions
- `product.js`: Product-related functions

---

## User Workflows

### **1. Farmer Registration**
```
Register.js (User selects "Farmer")
  ↓
Calls Farmer.registerFarmer()
  ↓
Smart contract stores farmer data
  ↓
Admin verifies farmer
  ↓
Farmer can now add raw products
```

### **2. Manufacturer Registration**
```
Register.js (User selects "Manufacturer")
  ↓
Calls Manufacturer.register()
  ↓
Manufacturer adds raw materials from verified farmers
  ↓
Manufacturer launches finished product
  ↓
Product appears in marketplace
```

### **3. Product Tracking**
```
Products/index.js (View all products)
  ↓
Click product → Products/product.js
  ↓
Shows:
  - Product details
  - Manufacturer info
  - Raw materials used
  - Transaction history
  - Reviews
```

### **4. Product Transfer**
```
Current owner initiates transfer
  ↓
Calls Product.transferProduct()
  ↓
New owner receives product
  ↓
Transaction recorded on blockchain
  ↓
Both parties can see updated ownership
```

### **5. Review System**
```
Consumer receives product
  ↓
Calls Product.addReview()
  ↓
Review stored immutably on blockchain
  ↓
Cannot be deleted or modified
  ↓
Affects product rating
```

---

## Data Flow Example: Tracking Milk

### **Step 1: Farmer Registers**
```
Frontend: Register.js
  ↓ User enters: Name, Location, Raw Products (Milk)
  ↓
Smart Contract: Farmer.registerFarmer()
  ↓ Stores in _farmerRawProducts mapping
  ↓
Blockchain: Transaction recorded
```

### **Step 2: Admin Verifies**
```
Frontend: Admin/VerifyFarmer.js
  ↓ Admin clicks "Verify"
  ↓
Smart Contract: Stakeholder.verify()
  ↓ Sets isVerified = true
  ↓
Blockchain: Verification recorded
```

### **Step 3: Manufacturer Adds Raw Material**
```
Frontend: Manufacturers/index.js
  ↓ Manufacturer adds "Milk" from verified farmer
  ↓
Smart Contract: Manufacturer.addRawProduct()
  ↓ Verifies farmer is legitimate
  ↓ Stores supplier information
  ↓
Blockchain: Raw material sourcing recorded
```

### **Step 4: Manufacturer Launches Product**
```
Frontend: Modals/LaunchProduct.js
  ↓ Manufacturer creates "Dairy Milk" (ID: 12091)
  ↓
Smart Contract: Product.add()
  ↓ Creates Item struct
  ↓ Links to manufacturer
  ↓ Sets currentOwner = manufacturer
  ↓
Blockchain: Product created with full history
```

### **Step 5: Retailer Purchases**
```
Frontend: Products/product.js
  ↓ Retailer clicks "Buy"
  ↓
Smart Contract: Product.transferProduct()
  ↓ Verifies manufacturer owns it
  ↓ Updates currentOwner = retailer
  ↓ Records transaction
  ↓
Blockchain: Ownership transfer recorded
```

### **Step 6: Consumer Verifies**
```
Frontend: Products/product.js
  ↓ Consumer scans QR or searches product ID
  ↓
Smart Contract: Product.getProductHistory()
  ↓ Returns all transactions
  ↓ Shows: Farmer → Manufacturer → Retailer → Consumer
  ↓
Frontend: Displays complete supply chain
```

---

## Key Technologies

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Blockchain** | Solidity 0.8.0 | Smart contracts |
| **Network** | Ethereum/Ganache | Blockchain network |
| **Frontend** | React.js | User interface |
| **Web3** | Web3.js | Blockchain interaction |
| **Wallet** | MetaMask | User authentication |
| **Storage** | NFT.storage | Off-chain image storage |
| **Testing** | Truffle/Chai | Smart contract testing |
| **Deployment** | Truffle | Contract deployment |

---

## How to Run

### **Smart Contracts**
```bash
cd src/Smart-Contract
npm install -g truffle
truffle compile
truffle migrate --reset
truffle test
```

### **Frontend**
```bash
npm install
npm start
```

### **Requirements**
- Node.js >= 10.16
- Ganache (local blockchain)
- MetaMask (browser extension)


