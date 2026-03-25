# MiniProject-BudgetAI
Mini Project budget AI
# Budget Tracker - Refactored Architecture

## 📁 Project Structure

```
src/
├── redux/
│   ├── store.js              # Redux store with persistence
│   ├── authSlice.js          # Authentication state management
│   └── budgetSlice.js        # Budget state management (NEW)
├── hooks/
│   ├── useBudgetValidation.js    # Validation logic (NEW)
│   └── useBudgetAPI.js           # API calls and data management (NEW)
├── components/
│   └── AddTransactionSimplified.jsx  # Simplified component (NEW)
└── db-clean.json             # Cleaned database structure (NEW)
```

## 🎯 Key Improvements

### 1. **Separation of Concerns**
- **Redux Slices**: Separate slices for auth and budget state
- **Custom Hooks**: Validation and API logic extracted from component
- **Clean DB Structure**: Proper separation of budget limits and category budgets

### 2. **Simplified Data Flow**
```
User Action → Component → Custom Hook → API Call → Redux Action → State Update → UI Update
```

### 3. **Better Database Structure**

**Old Structure Issues:**
- `transactions` table used for both category budgets AND transactions
- Duplicate budget entries with inconsistent IDs
- Mixed data types (strings and numbers)

**New Structure:**
```json
{
  "budgets": [           // User's total income and allotted budget
    {
      "id": "budget-{userId}-{timestamp}",
      "userId": "b132",
      "MaxBudget": 50000,
      "Balance": 30000,
      "Activate": true
    }
  ],
  "categoryBudgets": [   // NEW: Category-specific budget allocations
    {
      "id": "catbudget-{userId}-{catid}-{timestamp}",
      "userId": "b132",
      "catid": "101",
      "amount": 8000
    }
  ],
  "userTransactions": [  // Actual expense transactions
    {
      "id": "tx-{userId}-{timestamp}",
      "userId": "b132",
      "catid": "101",
      "amount": 1500,
      "type": "expense",
      "date": "2026-03-20",
      "month": "2026-03"
    }
  ]
}
```

## 🚀 Setup Instructions

### 1. Update Database Structure

Replace your `db.json` with the new `db-clean.json`:

```bash
cp db-clean.json db.json
```

### 2. Install Dependencies (if not already done)

```bash
npm install @reduxjs/toolkit react-redux
```

### 3. Update Redux Store

Replace your existing `store.js` with the new version that includes `budgetReducer`.

### 4. File Organization

```bash
# Create hooks directory
mkdir -p src/hooks

# Move files
mv budgetSlice.js src/redux/
mv useBudgetValidation.js src/hooks/
mv useBudgetAPI.js src/hooks/
mv AddTransactionSimplified.jsx src/components/
```

### 5. Update Imports

In your main app file:

```javascript
import { Provider } from 'react-redux';
import store from './redux/store';
import AddTransactionSimplified from './components/AddTransactionSimplified';

function App() {
  return (
    <Provider store={store}>
      <AddTransactionSimplified />
    </Provider>
  );
}
```

## 📝 How It Works

### 1. **Budget State Management (budgetSlice.js)**

```javascript
const budget = useSelector(state => state.budget);
// Access: budget.totalIncome, budget.allottedBudget, budget.categoryBudgets, etc.
```

### 2. **Validation Hook (useBudgetValidation.js)**

```javascript
const { validateBudgetLimits, validateCategoryBudgets, validateTransaction } = useBudgetValidation();

// Use anywhere you need validation
const error = validateBudgetLimits(income, budget);
if (error) {
  // Handle error
}
```

### 3. **API Hook (useBudgetAPI.js)**

```javascript
const { loadBudgetData, saveBudgetLimits, saveCategoryBudgets, saveTransaction } = useBudgetAPI(userId);

// Load data on mount
useEffect(() => {
  loadBudgetData();
}, [userId]);

// Save budget
const result = await saveBudgetLimits(income, budget, budgetId);
if (result.success) {
  // Handle success
}
```

## ✅ Validation Rules

### Budget Limits
- Total income must be > 0
- Allotted budget must be > 0
- Allotted budget cannot exceed total income

### Category Budgets
- No negative values allowed
- Sum of all category budgets must equal allotted budget
- Must set main budget limits before setting category budgets

### Transactions
- Must select a category
- Amount must be > 0
- Amount cannot exceed remaining category budget
- Must have category budget set first

## 🔄 Data Flow Example

### Setting Budget Limits

```javascript
// 1. User enters values in modal
setLimitInputs({ max: "50000", exp: "30000" });

// 2. Validation runs
const error = validateBudgetLimits("50000", "30000");
if (error) return;

// 3. API call
const result = await saveBudgetLimits("50000", "30000", budgetId);

// 4. Redux state updates automatically
// budget.totalIncome = 50000
// budget.allottedBudget = 30000

// 5. Component re-renders with new values
```

## 🎨 Component Simplification

**Before:** 500+ lines with mixed concerns
**After:** 400 lines with clear separation:
- UI rendering
- Event handlers
- Custom hooks for business logic
- Redux for state management

## 🐛 Common Issues & Solutions

### Issue: Budget not saving
**Solution:** Check that `userId` is properly set in Redux auth state

### Issue: Category budgets don't match total
**Solution:** Validation prevents saving until sum equals allotted budget

### Issue: Transaction fails to save
**Solution:** Ensure category budgets are set before adding transactions

## 📊 Redux DevTools

Install Redux DevTools to inspect state:

```bash
# Chrome Extension
https://chrome.google.com/webstore/detail/redux-devtools/
```

View state changes in real-time:
- Budget limits updates
- Category budget changes
- Transaction additions
- Loading states

## 🔐 Local Storage Persistence

Both auth and budget state are automatically persisted to localStorage:
- `budget-ai-auth`: User authentication
- `budget-ai-budget`: Budget limits and category budgets (transactions excluded)

## 🧪 Testing the Refactored Code

1. **Start JSON Server:**
```bash
json-server --watch db-clean.json --port 2512
```

2. **Test Flow:**
   - Login with existing user
   - Set income and budget limits
   - Set category budgets (must sum to allotted budget)
   - Add transactions
   - Verify data persists on page reload

## 💡 Benefits of Refactoring

1. **Maintainability:** Clear separation of concerns
2. **Testability:** Validation and API logic can be tested independently
3. **Reusability:** Hooks can be used in other components
4. **Type Safety:** Better structure for adding TypeScript later
5. **Performance:** Redux prevents unnecessary re-renders
6. **Debugging:** Redux DevTools for state inspection
7. **Scalability:** Easy to add new features

## 🚨 Migration Checklist

- [ ] Backup existing db.json
- [ ] Replace with db-clean.json
- [ ] Add budgetSlice to Redux store
- [ ] Create hooks directory
- [ ] Copy validation and API hooks
- [ ] Update component imports
- [ ] Test budget limits saving
- [ ] Test category budgets saving
- [ ] Test transaction creation
- [ ] Verify localStorage persistence
- [ ] Test with multiple users

## 📞 Support

If you encounter issues:
1. Check Redux state in DevTools
2. Verify API endpoints are accessible
3. Check browser console for errors
4. Ensure db.json structure matches db-clean.json
