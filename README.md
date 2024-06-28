Frontend:

npx create-react-app blog-app
cd blog-app
npm install @mui/material @emotion/react @emotion/styled redux react-redux quill draft-js

App.js:
import React from 'react';
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import HomePage from './components/HomePage';
import BlogDetailsPage from './components/BlogDetailsPage';
import CategoryPage from './components/CategoryPage';
import ThemeSwitcher from './components/ThemeSwitcher';

const App = () => {
  return (
    <Router>
      <ThemeSwitcher />
      <Switch>
        <Route exact path="/" component={HomePage} />
        <Route path="/blog/:id" component={BlogDetailsPage} />
        <Route path="/category/:name" component={CategoryPage} />
      </Switch>
    </Router>
  );
}

export default App;

Redux Setup:

// store.js
import { createStore, combineReducers, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import blogReducer from './reducers/blogReducer';
import userReducer from './reducers/userReducer';

const rootReducer = combineReducers({
  blogs: blogReducer,
  user: userReducer,
});

const store = createStore(rootReducer, applyMiddleware(thunk));

export default store;


Rich Text Editor Integration:

// components/RichTextEditor.js
import React, { useState } from 'react';
import ReactQuill from 'react-quill';
import 'react-quill/dist/quill.snow.css';

const RichTextEditor = ({ value, onChange }) => {
  return (
    <ReactQuill value={value} onChange={onChange} />
  );
};

export default RichTextEditor;

Theme Switcher:

// components/ThemeSwitcher.js
import React, { useState } from 'react';
import { createTheme, ThemeProvider, CssBaseline } from '@mui/material';
import Switch from '@mui/material/Switch';

const ThemeSwitcher = () => {
  const [darkMode, setDarkMode] = useState(false);

  const theme = createTheme({
    palette: {
      mode: darkMode ? 'dark' : 'light',
    },
  });

  return (
    <ThemeProvider theme={theme}>
      <CssBaseline />
      <Switch checked={darkMode} onChange={() => setDarkMode(!darkMode)} />
    </ThemeProvider>
  );
};

export default ThemeSwitcher;

Home Page Component:

// components/HomePage.js
import React, { useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { fetchBlogs } from '../actions/blogActions';
import { Link } from 'react-router-dom';

const HomePage = () => {
  const dispatch = useDispatch();
  const blogs = useSelector(state => state.blogs.items);

  useEffect(() => {
    dispatch(fetchBlogs());
  }, [dispatch]);

  return (
    <div>
      <h1>Featured Blogs</h1>
      <ul>
        {blogs.map(blog => (
          <li key={blog.id}>
            <Link to={/blog/${blog.id}}>
              <h2>{blog.title}</h2>
              <p>{blog.excerpt}</p>
              <p>{new Date(blog.createdAt).toDateString()}</p>
            </Link>
          </li>
        ))}
      </ul>
    </div>
  );
};

export default HomePage;



Backend:

mkdir backend
cd backend
npm init -y
npm install express mongoose bcryptjs jsonwebtoken

Server.js:

const express = require('express');
const mongoose = require('mongoose');
const bodyParser = require('body-parser');

const app = express();
app.use(bodyParser.json());

// Connect to MongoDB
mongoose.connect('mongodb://localhost/blog-app', { useNewUrlParser: true, useUnifiedTopology: true });

// Routes
const blogRoutes = require('./routes/blogRoutes');
const userRoutes = require('./routes/userRoutes');

app.use('/api/blogs', blogRoutes);
app.use('/api/users', userRoutes);

app.listen(5000, () => {
  console.log('Server running on port 5000');
});

Blog Schema:

const mongoose = require('mongoose');

const BlogSchema = new mongoose.Schema({
  title: { type: String, required: true },
  content: { type: String, required: true },
  author: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  createdAt: { type: Date, default: Date.now },
});

module.exports = mongoose.model('Blog', BlogSchema);

User Schema:

const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

const UserSchema = new mongoose.Schema({
  username: { type: String, required: true, unique: true },
  password: { type: String, required: true },
});

UserSchema.pre('save', async function (next) {
  if (!this.isModified('password')) return next();
  const salt = await bcrypt.genSalt(10);
  this.password = await bcrypt.hash(this.password, salt);
  next();
});

module.exports = mongoose.model('User', UserSchema);

Blog Routes:

const express = require('express');
const router = express.Router();
const Blog = require('../models/Blog');
const { authenticate } = require('../middleware/auth');

router.get('/', async (req, res) => {
  const blogs = await Blog.find().populate('author', 'username');
  res.json(blogs);
});

router.post('/', authenticate, async (req, res) => {
  const { title, content } = req.body;
  const blog = new Blog({ title, content, author: req.user.id });
  await blog.save();
  res.json(blog);
});

router.get('/:id', async (req, res) => {
  const blog = await Blog.findById(req.params.id).populate('author', 'username');
  res.json(blog);
});

router.put('/:id', authenticate, async (req, res) => {
  const { title, content } = req.body;
  const blog = await Blog.findById(req.params.id);
  if (blog.author.toString() !== req.user.id) {
    return res.status(403).json({ message: 'Forbidden' });
  }
  blog.title = title;
  blog.content = content;
  await blog.save();
  res.json(blog);
});

router.delete('/:id', authenticate, async (req, res) => {
  const blog = await Blog.findById(req.params.id);
  if (blog.author.toString() !== req.user.id) {
    return res.status(403).json({ message: 'Forbidden' });
  }
  await blog.remove();
  res.json({ message: 'Blog deleted' });
});

module.exports = router;

User Routes:

const express = require('express');
const router = express.Router();
const User = require('../models/User');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcryptjs');

router.post('/register', async (req, res) => {
  const { username, password } = req.body;
  const user = new User({ username, password });
  await user.save();
  res.json(user);
});

router.post('/login
