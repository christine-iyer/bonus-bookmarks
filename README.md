# Building Full-Stack App

```
npm run build
```
## Back End First

### Dependancies, Settings, Connections

```
npm i express mongoose dotenv morgan serve-favicon
```

```
mkdir models controllers routes config
```
```
touch .env .env-example server.js
```
**in .gitignore** add .env to the list of other .env files. 

```
# misc
.DS_Store
.env
.env.local
.env.development.local
.env.test.local
.env.production.local
```

**in server.js**

```
require('dotenv').config()
require('./config/database')
```

```
cd config
touch database.js
```

**in database.js**

```
const mongoose = require('mongoose');

mongoose.connect(process.env.MONGO_URI);

const db = mongoose.connection;
db.on('connected', function () {
     console.log(`Connected to ${db.name} at ${db.host}:${db.port}`)
})
```

**in .env**

```
MONGO_URI=mongodb+srv://edithbird:<password>@cluster0.w3guwmj.mongodb.net/<clustername>?retryWrites=true&w=majority
SECRET=thisisasecret
```

### Connect Database and Create App Object

**in server.js**

```
require('dotenv').config()
require('./config/database')
const express = require('express')
const path = require('path')
const favicon = require('serve-favicon')
const logger = require('morgan')
const PORT = process.env.PORT || 3001

const app = express()
```

### Add Middleware

**in server.js**

```
app.use(express.json())
app.use((req,res,next) => {
res.locals.data = {}
next()
})
app.use(logger('dev'))
app.use(favicon(path.join(__dirname, 'build', 'favicon.ico')))
app.use(express.static(path.join(__dirname, 'build')))
app.use('/api/placeholders', require('./routes/api/placeholders'))

app.get('/api/test', (req,res) => {
     res.join({'eureka': 'found it'})
})
app.get('*', (req, res) => {
     res.sendFile(path.join(__dirname, 'build', 'index.html'))
})

app.listen(PORT, ()=> {
     console.log(`I am listening on ${PORT}`)
})
```

### Models, Controllers, Routes

```
const {model, Schema} = require('mongoose')
const placeholderSchema = new Schema ({
     title: {required: true, type: String},
    completed: { required: true, type: String}
}, {timestamps:true
})
const Placeholder = model('Placeholder', placeholderSchema)
module.exports = Placeholder
```
**create placeholders.js in controllers/api**

```
const Placeholder = require('../../models/placeholder')


module.exports = {
     create, 
     indexComplete, 
     indexNotComplete,
     show,
     update, 
     destroy, 
     jsonPlaceholders, 
     jsonPlaceholder
}

//jsonPlaceholder, jsonPlaceholders
function jsonPlaceholder(req,res){
     res.json(res.locals.data.placeholder)
}
function jsonPlaceholders(req, res) {
     res.json(res.locals.data.placeholders)
}

//create
async function create(req, res, next){
     try {
          const placeholder = await Placeholder.create(req.body)
          console.log(placeholder)
          res.locals.data.placeholder = placeholder
          next()
     } catch (error) {
          res.status(400).json({ msg: error.message })
          
     }
}
//read- index, show
async function indexComplete(req,res,next) {
     try {
          const placeholders = await Placeholder.find({ completed:true })
          res.locals.data.placeholders = placeholders
          next()
     } catch (error) {
          res.status(400).json({ msg: error.message })
          
     }
}

async function indexNotComplete(req,res,next) {
     try {
          const placeholders = await Placeholder.find({ completed:false })
          res.locals.data.placeholders = placeholders
          next()
     } catch (error) {
          res.status(400).json({ msg: error.message })
          
     }
}
//show
async function show (req, res, next) {
     try {
          const placeholders = await Placeholder.findById(req.params.id)
          res.locals.data.placeholder = placeholder
          next()
     } catch (error) {
          res.status(400).json({ msg: error.message })
     }
}
//update
async function update (req, res, next) {
     try {
          const placeholder = await Placeholder.findByIdAndUpdate(req.params.id, req.body, { new : true }
               )
          res.locals.data.placeholder = placeholder
          next()
     } catch (error) {
          res.status(400).json({ msg: error.message })
     }
}
//destroy
async function destroy (req, res, next) {
     try {
          const placeholder = await Placeholder.findByIdAndDelete(req.params.id)
          res.locals.data.placeholder = placeholder
          next()
     } catch (error) {
          res.status(400).json({ msg: error.message })
     }
}
```

**create placeholders.js in routes/api**

```
const express = require('express')
const router = express.Router()
const placeholderCtrl = require('../../controllers/api/placeholders')

// Index /api/placeholders
router.get('/', bookmarkCtrl.indexNotComplete, placeholderCtrl.jsonPlaceholders)
//Index /api/placeholders/completed
router.get('/completed', placeholderCtrl.indexComplete,  placeholderCtrl.jsonPlaceholders)
//Delete /api/placeholders/:id
router.delete('/:id', placeholderCtrl.destroy, placeholderCtrl.jsonPlaceholder)
//Update /api/placeholders/:id
router.put('/:id', placeholderCtrl.update, placeholderCtrl.jsonPlaceholder)
//Create /api/placeholders
router.post('/', placeholderCtrl.create, placeholderCtrl.jsonPlaceholder)
//Show /api/placeholders/:id
router.get('/:id', placeholderCtrl.show, placeholderCtrl.jsonPlaceholder)

module.exports = router
```

**run server in dev mode***

```npm i nodemon```

**in package.json, replace...**

```
"scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
```

**with** 

```
"scripts": {
    "start": "node server.js", 
    "dev": "nodemon server.js",
    "react": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
  "proxy": "http://localhost:3001/",
```          
**test routes in postman**


## Restful Routes and API testing

| #  |Action   | URL |  HTTP Verb |  JSX file name |  mongoose |
|---|---|---|---|---|---|
|  1 | Index  |  /logs/ |  GET |  Index.jsx | Log.find  |
|   2|  Show | /logs/:id  |  GET | Show.jsx   |  Log.findById|
|  3 |  New |  /logs/new | GET  |  New.jsx |  none |
|   4|  Create | /logs  |  POST |   none| Log.create(req.body)  |
|   5| Edit  | /logs/:id/edit  | GET  | Edit.jsx  |  Log.findById |
|   6|  Update | /logs/:id   |  PUT|  none |  Log.findByIdAndUpdate |
|   7|  Destroy | /logs/:id   |  DELETE|  none |  Log.findByIdAndDelete |


### Routes for Postman 

example '63a3cf5e8318a07510819fec'

### User Login

POST localhost:3003/api/users/login

```js
router.post('/login', dataController.login, apiController.auth)
```

### Check user

POST localhost:3003/api/users

```js
router.post('/', dataController.create, apiController.auth)
```

### Check Token

GET localhost:3003/api/users/check-token

```js
router.get('/check-token', ensureLoggedIn, checkToken)
```



### Cart

GET localhost:3003/api/orders/cart

```js
router.get('/cart', ordersCtrl.cart);
```

### History

GET localhost:3003/api/orders/history

```js
router.get('/history', ordersCtrl.history);
```


### Cart

POST localhost:3003/api/orders/cart/items/63a3cf5e8318a07510819fec

```js
router.post('/cart/items/:id', ordersCtrl.addToCart);
```


### Checkout

POST localhost:3003/api/orders/cart/checkout

```js
router.post('/cart/checkout', ordersCtrl.checkout);
```


### Check Items in Cart

PUT localhost:3003/api/orders/cart/qty

```js
router.put('/cart/qty', ordersCtrl.setItemQtyInCart);
```


### INDEX

GET localhost:3003/api/items

```js
router.get('/', itemsCtrl.index);
```

### SHOW

GET localhost:3003/api/items/63a3cf5e8318a07510819fec

```js
router.get('/:id', itemsCtrl.show);
```



             

```npm run dev```

# Front End

**delete src folder and remake**

```
mkdir src
cd src
touch App.js index.js styles.css
```

**in index.js**

```
import './styles.css'
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client'
import App from './App'

const root = createRoot(document.getElementById('root'))

root.render(
    <StrictMode>
        <App/>
    </StrictMode>
)
```
**in src**

```mkdir BlogTitle BlogAuthor BlogBody
```
**in each folder, create a .js and .module.scss**
**BlogTitle/BlogTitle.js**

```
import { useState } from 'react';

const BlogTitle = () => {
	const [title, setTitle] = useState('');

	return (
		<div className="create">
			<h2>Blog</h2>
			<form>
				<label>Blog Title</label>
				<input
					type="text"
					required
					value={title}
					onChange={(e) => setTitle(e.target.value)}
				/>

				<button>Add Blog</button>
				<p>{title}</p>
			</form>
		</div>
	);
};
export default BlogTitle;
```

**BlogAuthor/BlogAuthor.js**

```
import { useState } from 'react';

const BlogAuthor = () => {
	const [author, setAuthor] = useState('luigi');

	return (
		<div className="create">
			<h2>Blog</h2>
			<form>
				<label>Blog Author</label>
				<select value={author} onChange={(e) => setAuthor(e.target.value)}>
					<option value="mario">mario</option>
					<option value="yoshi">yoshi</option>
					<option value="luigi">luigi</option>
				</select>
				<button>Add Blog</button>

				<p>{author}</p>
			</form>
		</div>
	);
};
export default BlogAuthor;
```

**BlogAuthor/BlogBody.js**

```
import { useState } from 'react';

const BlogBody = () => {
	const [body, setBody] = useState('');

	return (
		<div className="create">
			<h2>Blog</h2>
			<form>
				<label> Blog Body</label>
				<textarea
					required
					value={body}
					onChange={(e) => setBody(e.target.value)}></textarea>

				<button>Add Blog</button>

				<p>{body}</p>
			</form>
		</div>
	);
};
export default BlogBody;
```
**in Create/Create.js**

```
import { useState } from 'react';

const Create = () => {
	const [title, setTitle] = useState('');
	const [body, setBody] = useState('');
	const [author, setAuthor] = useState('luigi');

	return (
		<div className="create">
			<h2>Blog</h2>
			<form>
				<label>Blog Title</label>
				<input
					type="text"
					required
					value={title}
					onChange={(e) => setTitle(e.target.value)}
				/>
				<label> Blog Body</label>
				<textarea
					required
					value={body}
					onChange={(e) => setBody(e.target.value)}></textarea>
				<label>Blog Author</label>
				<select value={author} onChange={(e) => setAuthor(e.target.value)}>
					<option value="mario">mario</option>
					<option value="yoshi">yoshi</option>
					<option value="luigi">luigi</option>
				</select>
				<button>Add Blog</button>
				<p>{title}</p>
				<p>{body}</p>
				<p>{author}</p>
			</form>
		</div>
	);
};
export default Create;
```

**in App.js**

```
import Create from './Create/Create';

export default function App() {
	return (
		<div className="App">
          <Create />
		</div>
	);
}
```

### Pages and Authentication



```npm i -g scss```





```npm run react```



