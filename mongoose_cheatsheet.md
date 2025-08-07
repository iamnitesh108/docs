# Mongoose Cheatsheet for Web Applications

## Installation & Setup

```bash
npm install mongoose
```

```javascript
const mongoose = require('mongoose');

// Connect to MongoDB
mongoose.connect('mongodb://localhost:27017/myapp', {
  useNewUrlParser: true,
  useUnifiedTopology: true
});

// Connection events
mongoose.connection.on('connected', () => {
  console.log('Connected to MongoDB');
});

mongoose.connection.on('error', (err) => {
  console.log('MongoDB connection error:', err);
});

mongoose.connection.on('disconnected', () => {
  console.log('Disconnected from MongoDB');
});
```

## Schema Definition

```javascript
const mongoose = require('mongoose');
const { Schema } = mongoose;

// Basic Schema
const userSchema = new Schema({
  name: String,
  email: String,
  age: Number,
  createdAt: {
    type: Date,
    default: Date.now
  }
});

// Schema with Validation
const productSchema = new Schema({
  name: {
    type: String,
    required: true,
    trim: true,
    maxlength: 100
  },
  price: {
    type: Number,
    required: true,
    min: 0
  },
  category: {
    type: String,
    enum: ['electronics', 'clothing', 'books'],
    required: true
  },
  tags: [String],
  inStock: {
    type: Boolean,
    default: true
  }
});

// Schema with References
const orderSchema = new Schema({
  user: {
    type: Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  products: [{
    product: {
      type: Schema.Types.ObjectId,
      ref: 'Product'
    },
    quantity: {
      type: Number,
      default: 1
    }
  }],
  totalAmount: Number,
  status: {
    type: String,
    enum: ['pending', 'shipped', 'delivered'],
    default: 'pending'
  }
});
```

## Common Schema Types

```javascript
const exampleSchema = new Schema({
  // String
  name: String,
  title: { type: String, required: true },
  
  // Number
  age: Number,
  price: { type: Number, min: 0, max: 1000 },
  
  // Date
  createdAt: { type: Date, default: Date.now },
  updatedAt: Date,
  
  // Boolean
  isActive: { type: Boolean, default: true },
  
  // Array
  tags: [String],
  scores: [Number],
  
  // Object/Subdocument
  address: {
    street: String,
    city: String,
    zipCode: String
  },
  
  // ObjectId (Reference)
  userId: { type: Schema.Types.ObjectId, ref: 'User' },
  
  // Mixed (any type)
  metadata: Schema.Types.Mixed,
  
  // Buffer
  data: Buffer
});
```

## Schema Methods & Virtuals

```javascript
// Instance Methods
userSchema.methods.getFullName = function() {
  return `${this.firstName} ${this.lastName}`;
};

// Static Methods
userSchema.statics.findByEmail = function(email) {
  return this.findOne({ email });
};

// Virtual Properties
userSchema.virtual('fullName').get(function() {
  return `${this.firstName} ${this.lastName}`;
});

// Pre-save Middleware
userSchema.pre('save', function(next) {
  this.updatedAt = Date.now();
  next();
});

// Post-save Middleware
userSchema.post('save', function(doc) {
  console.log('User saved:', doc._id);
});
```

## Creating Models

```javascript
const User = mongoose.model('User', userSchema);
const Product = mongoose.model('Product', productSchema);
const Order = mongoose.model('Order', orderSchema);
```

## CRUD Operations

### Create (Insert)

```javascript
// Create single document
const user = new User({
  name: 'John Doe',
  email: 'john@example.com',
  age: 30
});

await user.save();

// Or use create method
const user2 = await User.create({
  name: 'Jane Doe',
  email: 'jane@example.com',
  age: 25
});

// Create multiple documents
await User.insertMany([
  { name: 'User 1', email: 'user1@example.com' },
  { name: 'User 2', email: 'user2@example.com' }
]);
```

### Read (Find)

```javascript
// Find all
const users = await User.find();

// Find with conditions
const adults = await User.find({ age: { $gte: 18 } });

// Find one
const user = await User.findOne({ email: 'john@example.com' });

// Find by ID
const user = await User.findById('60d5ecb54b24a1234567890a');

// Find with select (projection)
const users = await User.find().select('name email -_id');

// Find with sorting
const users = await User.find().sort({ age: -1 }); // Descending

// Find with pagination
const users = await User.find()
  .skip(10)
  .limit(5)
  .sort({ createdAt: -1 });

// Find with population
const orders = await Order.find().populate('user', 'name email');
const orders = await Order.find().populate({
  path: 'products.product',
  select: 'name price'
});
```

### Update

```javascript
// Update one document
await User.updateOne(
  { _id: userId },
  { $set: { age: 31 } }
);

// Update many documents
await User.updateMany(
  { age: { $lt: 18 } },
  { $set: { isMinor: true } }
);

// Find and update (returns updated document)
const user = await User.findByIdAndUpdate(
  userId,
  { age: 32 },
  { new: true } // Return updated document
);

// Find one and update
const user = await User.findOneAndUpdate(
  { email: 'john@example.com' },
  { age: 33 },
  { new: true, upsert: true } // Create if doesn't exist
);
```

### Delete

```javascript
// Delete one document
await User.deleteOne({ _id: userId });

// Delete many documents
await User.deleteMany({ age: { $lt: 18 } });

// Find and delete (returns deleted document)
const deletedUser = await User.findByIdAndDelete(userId);

// Find one and delete
const deletedUser = await User.findOneAndDelete({ email: 'john@example.com' });
```

## Query Operators

### Comparison Operators

```javascript
// Equal
User.find({ age: 25 })

// Not equal
User.find({ age: { $ne: 25 } })

// Greater than
User.find({ age: { $gt: 18 } })

// Greater than or equal
User.find({ age: { $gte: 18 } })

// Less than
User.find({ age: { $lt: 65 } })

// Less than or equal
User.find({ age: { $lte: 65 } })

// In array
User.find({ age: { $in: [20, 25, 30] } })

// Not in array
User.find({ age: { $nin: [20, 25, 30] } })
```

### Logical Operators

```javascript
// AND (implicit)
User.find({ age: { $gte: 18 }, isActive: true })

// AND (explicit)
User.find({ $and: [{ age: { $gte: 18 } }, { isActive: true }] })

// OR
User.find({ $or: [{ age: { $lt: 18 } }, { age: { $gt: 65 } }] })

// NOT
User.find({ age: { $not: { $gte: 18 } } })

// NOR
User.find({ $nor: [{ age: { $lt: 18 } }, { isActive: false }] })
```

### Array Operators

```javascript
// All elements match
User.find({ tags: { $all: ['javascript', 'nodejs'] } })

// Array has specific size
User.find({ tags: { $size: 3 } })

// Element matches condition
User.find({ 'scores.0': { $gt: 80 } }) // First element > 80

// Any array element matches
User.find({ scores: { $elemMatch: { $gt: 80, $lt: 90 } } })
```

### String Operators

```javascript
// Regex match
User.find({ name: /john/i }) // Case insensitive

// Text search (requires text index)
User.find({ $text: { $search: "john doe" } })
```

## Aggregation Pipeline

```javascript
// Basic aggregation
const result = await User.aggregate([
  { $match: { age: { $gte: 18 } } },
  { $group: { _id: '$department', count: { $sum: 1 } } },
  { $sort: { count: -1 } }
]);

// Complex aggregation with lookup (join)
const ordersWithUsers = await Order.aggregate([
  {
    $lookup: {
      from: 'users',
      localField: 'user',
      foreignField: '_id',
      as: 'userInfo'
    }
  },
  { $unwind: '$userInfo' },
  {
    $project: {
      totalAmount: 1,
      'userInfo.name': 1,
      'userInfo.email': 1
    }
  }
]);

// Group with multiple operations
const stats = await Order.aggregate([
  {
    $group: {
      _id: '$status',
      count: { $sum: 1 },
      totalAmount: { $sum: '$totalAmount' },
      avgAmount: { $avg: '$totalAmount' },
      maxAmount: { $max: '$totalAmount' },
      minAmount: { $min: '$totalAmount' }
    }
  }
]);
```

## Validation

```javascript
// Built-in validators
const userSchema = new Schema({
  email: {
    type: String,
    required: [true, 'Email is required'],
    unique: true,
    lowercase: true,
    trim: true,
    match: [/^\S+@\S+\.\S+$/, 'Invalid email format']
  },
  age: {
    type: Number,
    min: [0, 'Age must be positive'],
    max: [120, 'Age must be realistic']
  },
  role: {
    type: String,
    enum: {
      values: ['user', 'admin', 'moderator'],
      message: 'Invalid role'
    }
  }
});

// Custom validator
userSchema.path('email').validate(async function(email) {
  const user = await User.findOne({ email });
  return !user || user._id.equals(this._id);
}, 'Email already exists');

// Validation on save
try {
  await user.save();
} catch (error) {
  if (error.name === 'ValidationError') {
    console.log('Validation errors:', error.errors);
  }
}
```

## Indexes

```javascript
// Create indexes
userSchema.index({ email: 1 }); // Ascending
userSchema.index({ createdAt: -1 }); // Descending
userSchema.index({ name: 'text' }); // Text index
userSchema.index({ email: 1, name: 1 }); // Compound index

// Unique index
userSchema.index({ email: 1 }, { unique: true });

// Partial index
userSchema.index(
  { email: 1 },
  { 
    unique: true,
    partialFilterExpression: { email: { $exists: true } }
  }
);
```

## Transactions

```javascript
// Using transactions
const session = await mongoose.startSession();

try {
  await session.withTransaction(async () => {
    const user = await User.create([{
      name: 'John',
      email: 'john@example.com'
    }], { session });

    const order = await Order.create([{
      user: user[0]._id,
      totalAmount: 100
    }], { session });
  });
} catch (error) {
  console.log('Transaction failed:', error);
} finally {
  await session.endSession();
}
```

## Middleware Hooks

```javascript
// Pre-save
userSchema.pre('save', function(next) {
  if (!this.isModified('password')) return next();
  // Hash password here
  next();
});

// Post-save
userSchema.post('save', function(doc) {
  console.log('Document saved:', doc._id);
});

// Pre-remove
userSchema.pre('remove', function(next) {
  // Clean up related documents
  Order.deleteMany({ user: this._id }).exec();
  next();
});

// Query middleware
userSchema.pre(/^find/, function(next) {
  this.select('-__v'); // Exclude version field
  next();
});
```

## Common Patterns

### Error Handling

```javascript
// Try-catch with async/await
try {
  const user = await User.findById(userId);
  if (!user) {
    throw new Error('User not found');
  }
  // Process user
} catch (error) {
  console.error('Error:', error.message);
}

// Validation error handling
user.save((error) => {
  if (error) {
    if (error.name === 'ValidationError') {
      Object.keys(error.errors).forEach(key => {
        console.log(error.errors[key].message);
      });
    }
  }
});
```

### Connection Management

```javascript
// Graceful shutdown
process.on('SIGINT', async () => {
  await mongoose.connection.close();
  console.log('MongoDB connection closed.');
  process.exit(0);
});

// Connection with options
mongoose.connect('mongodb://localhost:27017/myapp', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
  maxPoolSize: 10, // Maintain up to 10 socket connections
  serverSelectionTimeoutMS: 5000, // Keep trying to send operations for 5 seconds
  socketTimeoutMS: 45000, // Close sockets after 45 seconds of inactivity
});
```

### Environment Configuration

```javascript
// Using environment variables
const mongoURI = process.env.MONGODB_URI || 'mongodb://localhost:27017/myapp';
mongoose.connect(mongoURI);

// Different configs for different environments
const config = {
  development: 'mongodb://localhost:27017/myapp_dev',
  test: 'mongodb://localhost:27017/myapp_test',
  production: process.env.MONGODB_URI
};

mongoose.connect(config[process.env.NODE_ENV || 'development']);
```

## Best Practices

1. **Always use await/async or proper error handling**
2. **Create indexes for frequently queried fields**
3. **Use lean() for read-only operations to improve performance**
4. **Validate data both on client and server side**
5. **Use projections to limit returned fields**
6. **Implement proper connection error handling**
7. **Use transactions for operations that require atomicity**
8. **Keep schemas normalized when possible**
9. **Use virtuals for computed properties**
10. **Implement proper logging and monitoring**
