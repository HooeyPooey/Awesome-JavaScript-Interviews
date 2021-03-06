Lets take an example of a social network, one collection for users, and one for posts. In my research before doing any coding, I stumbled upon ``Model.populate()``, a Mongoose method that you can use to essentially link documents across collections.


### Step 1: Make your schemas
You need a schema for each collection. One for the users, and one for the posts those users are going to make.

```js

const UserSchema = new mongoose.Schema({
    username: String,
    posts: [{
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Post'
    }]
  })

const PostSchema = new mongoose.Schema({
    content: String,
    author: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User'
    }
  })

const Post = mongoose.model('Post', PostSchema, 'posts');
const User = mongoose.model('User', UserSchema, 'users');

module.exports = {
    User, Post,
  }

```

This tells Mongoose “Hey, I’m gonna be referencing other documents from other collections”. The next part of that property is the ref (Post or User in the above code). The ref tells Mongoose “Those docs are going to be in the Post or User collection.”

So in our User schema, we reference the Post collection, because we want the user to be tied to the things they post, and we want to be able to easily access those posts without having to create more queries.

After linking other collections in your schema using the appropriate type and ref, your actual stored data for that property will be another document’s _id. It will be stored as a string. This also works for an array of _ids.

So while your schema says this:

```js
const UserSchema = new mongoose.Schema({
    username: String,
    posts: [{
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Post'
    }]
  })

```
Your actual stored property should read something like this:

```js
{
  _id: 59ab1c92ea84486fb4ba9f28,
  username: JD,
  posts: [
    "59ab1b43ea84486fb4ba9ef0",
    "59ab1b43ea84486fb4ba9ef1"
  ]
}

```

Keep in mind that this is your stored document. We have not called .populate() on it yet.

#### Once it is called, it will go to the appropriate collection, search for those two _ids, and return your user, but now with an array of her actual posts. Let’s do that now.


```js
function getUserWithPosts(username) {
    return User.findOne({ username: username })
        .populate('posts').exec(err, posts) => {
            console.log("Populated User " + posts)
        })
}
```
.populate() needs a query to attach itself to, so we are using User.findOne() to find a user who matches the username we provide in the argument. This returns our user document. This is when .populate() takes over.

#### You’ll notice I am providing ‘posts’ to our .populate(). By providing the ‘posts’ argument, we’ve told .populate() what property in our user document we want it to work with. Calling .exec() just executes something once .populate() has done it’s thing. The log prints this:

```js
{
  _id: 59ab1c92ea84486fb4ba9f28,
  username: 'JD',
  posts:
    [
      {
        _id: 59ab1b43ea84486fb4ba9ef0,
        content: "Is it dark out?"
      },{
        _id: 59ab1b43ea84486fb4ba9ef1,
        content: "Hey anyone got a cup of sugar?"
      }
    ]
  }

```

Arrays of ObjectId refs works like this. Just call the populate method on the query and an array of documents will be returned in place of the ObjectIds.

And like magic, we have created a unified object using 2 schemas, 2 models, and 2 collections. All of the steps are important of course, but the thing that no other site made explicitly clear was that after setting up the ground work, you have to make sure you are pushing _ids into the field you will need populated later.


### Another implementation

Let’s pretend we’re building a social app, and we have two models: a User and a Post:

```js
var UserSchema = {
  _id: String,
  username: String
};

var PostSchema = {
  _id: String,
  user: {
    ref: 'User',
    type: String
  }

```

If you run this query: ``Post.find({}).populate('user').exec(callback)``, Mongoose will look at the field user in the post, see that it has a ref to the User model, and find that user by its _id


### Sources to read

[https://medium.com/@nicknauert/mongooses-model-populate-b844ae6d1ee7](https://mongoosejs.com/docs/2.7.x/docs/populate.html)

[https://mongoosejs.com/docs/2.7.x/docs/populate.html](https://mongoosejs.com/docs/2.7.x/docs/populate.html)