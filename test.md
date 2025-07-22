# Testing splid-js with Your Group

## Step 1: Install Node.js
1. Go to https://nodejs.org
2. Download the LTS (Long Term Support) version
3. Install it (just follow the installer prompts)
4. Open terminal/command prompt and verify: `node --version`

## Step 2: Create a Test Project
```bash
# Create a new folder
mkdir splid-test
cd splid-test

# Initialize a new Node.js project
npm init -y

# Install splid-js
npm install splid-js
```

## Step 3: Create Test Files

### Basic Connection Test (`test-connection.js`)
```javascript
import { SplidClient } from 'splid-js';

async function testConnection() {
  try {
    console.log('🔍 Testing connection to your group...');
    
    const splid = new SplidClient();
    const inviteCode = 'E7K TVE 7Y6';  // Your group code
    
    // Get your group
    const groupRes = await splid.group.getByInviteCode(inviteCode);
    console.log('✅ Group found:', groupRes.result.objectId);
    
    const groupId = groupRes.result.objectId;
    
    // Get group info
    const groupInfo = await splid.groupInfo.getOneByGroup(groupId);
    console.log('📋 Group name:', groupInfo.name);
    console.log('💰 Default currency:', groupInfo.defaultCurrencyCode);
    
    // Get members
    const members = await splid.person.getAllByGroup(groupId);
    console.log('👥 Members:');
    members.forEach(member => {
      console.log(`  - ${member.name} (${member.GlobalId})`);
    });
    
    // Get existing expenses
    const entries = await splid.entry.getAllByGroup(groupId);
    const expenses = entries.filter(e => !e.isPayment);
    console.log(`💸 Current expenses: ${expenses.length}`);
    
    return { groupId, members, groupInfo };
    
  } catch (error) {
    console.error('❌ Error:', error.message);
  }
}

testConnection();
```

### Expense Creation Test (`test-create-expense.js`)
```javascript
import { SplidClient } from 'splid-js';

async function testCreateExpense() {
  try {
    console.log('🧪 Testing expense creation...');
    
    const splid = new SplidClient();
    const inviteCode = 'E7K TVE 7Y6';
    
    // Get group info
    const groupRes = await splid.group.getByInviteCode(inviteCode);
    const groupId = groupRes.result.objectId;
    
    // Get members to use their IDs
    const members = await splid.person.getAllByGroup(groupId);
    console.log('Available members:');
    members.forEach((member, index) => {
      console.log(`  ${index}: ${member.name} (${member.GlobalId})`);
    });
    
    // ⚠️ IMPORTANT: Replace these indices with actual member positions
    const payerIndex = 0;  // Change this to who should pay
    const profiteerIndex1 = 0;  // Change this to who benefits
    const profiteerIndex2 = 1;  // Change this to who benefits (if exists)
    
    if (members.length < 2) {
      console.log('⚠️ Need at least 2 members to test expense splitting');
      return;
    }
    
    // Create a test expense
    const expense = await splid.entry.expense.create(
      {
        groupId,
        payers: [members[payerIndex].GlobalId],
        title: '🧪 Test Coffee (API Test)',
      },
      {
        amount: 5.50,
        profiteers: [
          members[profiteerIndex1].GlobalId,
          members[profiteerIndex2].GlobalId
        ],
      }
    );
    
    console.log('✅ Expense created successfully!');
    console.log('📄 Details:', {
      title: expense.title,
      amount: expense.items?.[0]?.AM || 'Unknown',
      payer: members[payerIndex].name
    });
    
    return expense;
    
  } catch (error) {
    console.error('❌ Error creating expense:', error.message);
    console.error('Full error:', error);
  }
}

testCreateExpense();
```

### Express.js REST API Test (`test-rest-api.js`)
```javascript
import express from 'express';
import { SplidClient } from 'splid-js';

const app = express();
app.use(express.json());

// Your create expense endpoint
app.post('/groups/:groupId/expenses', async (req, res) => {
  try {
    const { groupId } = req.params;
    const { title, payers, items } = req.body;
    
    console.log('🔍 Creating expense:', { title, payers, items });
    
    const splid = new SplidClient();
    
    // Create expense using splid-js
    const expense = await splid.entry.expense.create(
      {
        groupId,
        payers,
        title,
      },
      items
    );
    
    console.log('✅ Expense created via REST API');
    
    res.status(201).json({
      success: true,
      data: expense
    });
    
  } catch (error) {
    console.error('❌ REST API Error:', error.message);
    res.status(500).json({
      error: 'Failed to create expense',
      details: error.message
    });
  }
});

// Test endpoint to get group info
app.get('/groups/by-invite/:inviteCode', async (req, res) => {
  try {
    const { inviteCode } = req.params;
    const splid = new SplidClient();
    
    const groupRes = await splid.group.getByInviteCode(inviteCode);
    const groupId = groupRes.result.objectId;
    
    const [groupInfo, members] = await Promise.all([
      splid.groupInfo.getOneByGroup(groupId),
      splid.person.getAllByGroup(groupId)
    ]);
    
    res.json({
      success: true,
      data: {
        groupId,
        name: groupInfo.name,
        members: members.map(m => ({
          id: m.GlobalId,
          name: m.name,
          initials: m.initials
        }))
      }
    });
    
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

const PORT = 3000;
app.listen(PORT, () => {
  console.log(`🚀 REST API server running on http://localhost:${PORT}`);
  console.log(`📋 Test your group: GET /groups/by-invite/E7K%20TVE%207Y6`);
  console.log(`💸 Create expense: POST /groups/{groupId}/expenses`);
});
```

## Step 4: Update package.json
Add this to your `package.json`:
```json
{
  "type": "module",
  "scripts": {
    "test-connection": "node test-connection.js",
    "test-create": "node test-create-expense.js",
    "test-api": "node test-rest-api.js"
  }
}
```

For REST API testing, also install Express:
```bash
npm install express
```

## Step 5: Run Tests

### Test 1: Basic Connection
```bash
npm run test-connection
```
This will show your group info and members.

### Test 2: Create Expense
```bash
npm run test-create
```
⚠️ **IMPORTANT**: First run the connection test to see member IDs, then update the indices in the create test!

### Test 3: REST API
```bash
npm run test-api
```
Then in another terminal/browser:
```bash
# Get group info
curl "http://localhost:3000/groups/by-invite/E7K%20TVE%207Y6"

# Create expense (replace GROUP_ID and MEMBER_IDS with real values)
curl -X POST "http://localhost:3000/groups/GROUP_ID/expenses" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "API Test Pizza",
    "payers": ["MEMBER_ID_1"],
    "items": {
      "amount": 12.50,
      "profiteers": ["MEMBER_ID_1", "MEMBER_ID_2"]
    }
  }'
```

## What Each Test Does

🔍 **Connection Test**: Verifies you can access your group and see members  
🧪 **Create Test**: Actually creates a test expense in your group  
🚀 **REST API Test**: Sets up HTTP endpoints you can call with curl/Postman

Start with the connection test to make sure everything works, then move to expense creation!