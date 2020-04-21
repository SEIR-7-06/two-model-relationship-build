# Creating A Relationship Between Two Models

## Lesson Objectives

1. Add Articles Array to Author Model
1. Display Authors on New Article Page
1. Creating a new Article Pushes a Copy Onto Author's Articles Array
1. Display Author With Link on Article Show Page
1. Display Author's Articles With Links On Author Show Page
1. Deleting an Article Updates An Author's Articles List
1. Updating an Article Updates An Author's Articles List
1. Deleting an Author Deletes The Associated Articles
1. Change Author When Editing an Article



### Numerical Categories for Relationships

### One-to-One

Each person has one brain, and each (living human) brain belongs to one person.

![one to one erd example](https://cloud.githubusercontent.com/assets/3254910/18140904/4d85c04e-6f6c-11e6-8301-c06bacff3dd3.png)

One-to-one relationships can sometimes just be modeled with simple attributes. A person and a brain are both complex enough that we might want to have their data in different models, with lots of different attributes on each.






### One-to-Many

Each leaf "belongs to" the one tree it grew from, and each tree "has many" leaves.

![one to many erd example](https://cloud.githubusercontent.com/assets/3254910/18182445/e4bddb6c-7044-11e6-9099-314b773724f3.png)


### Many-to-Many

Each student "has many" classes they attend, and each class "has many" students.


![many to many erd example](https://cloud.githubusercontent.com/assets/3254910/18140903/4c56c3ee-6f6c-11e6-9b6d-4c6ffae81323.png)


#### Entity Relationship Diagrams

Entity relationship diagrams (ERDs) represent information about the numerical relationships between data, or entities.

![entity relationship diagram example](https://cloud.githubusercontent.com/assets/3254910/18141666/439d9392-6f6f-11e6-953f-c91415b85f3f.png)


Note: In the example above, all of the Item1, Item2, Item3 under each heading are standing in for attributes.

[More guidelines for ERDs](http://docs.oracle.com/cd/A87860_01/doc/java.817/a81358/05_dev1.htm)

#### Check for Understanding

Come up with an example of related data.  Draw the ERD for your relationship, including a few attributes for each model.

### Association Categories for Mongoose

**Embedded Data** is directly nested *inside* of other data. Each record has a copy of the data.


It is often *efficient* to embed data because you don't have to make a separate request or a separate database query -- the first request or query gets you all the information you need.  


<img src="https://i.imgur.com/aMG36rT.png" width="60%">


**Referenced Data** is stored as an *id* inside other data. The id can be used to look up the information. All records that reference the same data look up the same copy.


It is usually easier to keep referenced records *consistent* because the data is only stored in one place and only needs to be updated in one place.  

![image](https://cloud.githubusercontent.com/assets/6520345/21190300/2c091f08-c1d6-11e6-89ed-0459874edf3a.png)
[Source: MongoDB docs](https://docs.mongodb.com/v3.2/tutorial/model-referenced-one-to-many-relationships-between-documents/)


While the question of one-to-one, one-to-many, or  many-to-many is often determined by real-world characteristics of a relationship, the decision to embed or reference data is a design decision.  

There are tradeoffs, such as between *efficiency* and *consistency*, depending on which one you choose.  

When using Mongo and Mongoose, though, many-to-many relationships often involve referenced associations, while one-to-many often involve embedding data.



### Relevant Documentation

1. [Populate](https://mongoosejs.com/docs/populate.html)

## Add Articles Array to Author Model

models/authors.js

```javascript
const Article = require('./models/articles.js')

const authorSchema = new mongoose.Schema({
  name: String,
  articles: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Article'
  }]
});
```

- `[]` lets the author schema know that each authors's `articles` attribute will hold an array.
- The object inside the `[]` describes what kind of elements the array will hold.
- Giving `type: Schema.Types.ObjectId` tells the schema the `articles` array will hold ObjectIds. That's the type of that unique `_id` that Mongo automatically generates for us (something like `55e4ce4ae83df339ba2478c6`).
- `ref: Article` tells the schema we will only be putting ObjectIds of  `Article` documents inside the `articles` array.

## Display Authors on New Article Page

Require the Author model in controllers/articles.js:

```javascript
const Author = require('../models/authors.js')
```

Find all Authors When Rendering New Page:

```javascript
router.get('/new', (req, res)=>{
    Author.find({}, (err, allAuthors)=>{
        res.render('articles/new.ejs', {
            authors: allAuthors
        });
    });
});
```

Create A Select Element in views/articles/new.ejs:

```html
<form action="/articles" method="post">
    <select name="authorId">
        <% for(let i = 0; i < authors.length; i++) { %>
            <option value="<%=authors[i]._id%>"><%=authors[i].name%></option>
        <% } %>
    </select><br/>
    <input type="text" name="title" /><br/>
    <textarea name="body"></textarea><br/>
    <input type="submit" value="Publish Article"/>
</form>
```

## Creating a new Article Pushes a Copy Onto Author's Articles Array

controllers/articles.js:

```javascript
Articles.create(req.body, (err, createdArticle) => {
        if(err){
          res.send(err);
        } else {
          Author.findById(req.body.authorId, (error, foundAuthor) => {
            console.log(foundAuthor, 'foundAuthor');
            foundAuthor.articles.push(createdArticle);
            foundAuthor.save((err, savedAuthor) => {
              console.log(savedAuthor, 'savedNewAuthor')
              res.redirect('/articles');
            })
          })
        }
      })
```

**NOTE: req.body.authorId is ignored when creating Article due to Article Schema**

## Display Author With Link on Article Show Page

controllers/articles.js:

```javascript
// articles show route
// :id is going to be the articles id
router.get('/:id', (req, res)=>{
  // req.params.id is an article id
  Author.findOne({'articles': req.params.id})
    .populate(
        {
        path: 'articles',
        match: {_id: req.params.id}
        }) // this pulls in the articles document
    .exec((err, foundAuthor) => { //exec executes the query
      console.log(foundAuthor, ' this is found AUthorrrr')
      if(err){
        res.send(err);
      } else {
        res.render('articles/show.ejs', {
          author: foundAuthor,
          article: foundAuthor.articles[0]
        });
      }
    })
});
```

1. Line 1: We call a method to find only **one** `Author` document that matches the id: `req.params.id` (Which is the article id.

1. Line 2: We ask the articles array within that `Author` document to fetch the actual `Article` document instead of just  its `ObjectId`.

1. Line 3: When we use `find` without a callback, then `populate`, like here, we can put a callback inside an `.exec()` method call. Technically we have made a query with `find`, but only executed it when we call `.exec()`.

views/articles/show.ejs:

```html
<h1><%=article.title%></h1>
<small>by: <a href="/authors/<%=author._id%>"><%=author.name%></a></small>
```

## Display Author's Articles With Links On Author Show Page

authors/show route
```javascript
router.get('/:id', (req, res) => {

  Authors.findById(req.params.id)
  .populate({path: 'articles'})
  .exec((err, foundAuthor) => {
    if(err) console.log(err);
      console.log(foundAuthor)

      res.render('authors/show.ejs', {
        author: foundAuthor
      });
  })
});


```

views/authors/show.ejs:

```html
<section>
    <h2>Articles Written By This Author:</h2>
    <ul>
        <% for(let i = 0; i < author.articles.length; i++){ %>
            <li><a href="/articles/<%=author.articles[i]._id%>"><%=author.articles[i].title%></a></li>
        <% } %>
    </ul>
</section>
```

## Deleting an Article Updates An Author's Articles List

controllers/articles.js

```javascript
router.delete('/:id', (req, res)=>{
  Article.findByIdAndRemove(req.params.id, (err, deletedArticle)=>{
    Author.findOne({'articles': req.params.id}, (err, foundAuthor) => {
         if(err){
            res.send(err);
          } else {
            foundAuthor.articles.remove(req.params.id);
            foundAuthor.save((err, updatedAuthor) => {
              console.log(updatedAuthor);
              res.redirect('/articles');
            })
          }
    })
  });
});
```


## Deleting an Author Deletes The Associated Articles

controllers/authors.js

```javascript
const Article = require('../models/articles.js');

//...farther down the file
router.delete('/:id', (req, res)=>{
	Authors.findByIdAndRemove(req.params.id, (err, deletedAuthor) => {
    if(err) {
      console.error(err);
      res.send("it didn't work check the console")
    }
    else {
      // we want to get all the article id's associated

      Articles.remove({
        _id: {
          $in: deletedAuthor.articles
        }
      }, (err, data) => {
        console.log(data, ' dat')
        res.redirect('/authors')
      });

    }
  })
});
```

## Change Author When Editing an Article

controllers/articles.js

```javascript
router.get('/:id/edit', (req, res)=>{
  Author.find({}, (err, allAuthors) => {
    Author.findOne({'articles': req.params.id})
    .populate({path: 'articles', match: {_id: req.params.id}})
    .exec((err, foundArticleAuthor) => {
        if(err){
          res.send(err);
        } else {
          res.render('articles/edit.ejs', {
            article: foundArticleAuthor.articles[0],
            authors: allAuthors,
            articleAuthor: foundArticleAuthor
          });
        }
    })
  })
});
```

views/articles/edit.ejs

```html
<form action="/articles/<%=article._id%>?_method=PUT" method="post">
    <select name="authorId">
        <% for(let i = 0; i < authors.length; i++) { %>
            <option
                value="<%=authors[i]._id%>"
                <% if(authors[i]._id.toString() === articleAuthor._id.toString()){ %>
                    selected
                <% } %>
                >
                <%=authors[i].name%>
            </option>
        <% } %>
    </select><br/>
    <input type="text" name="title" value="<%=article.title%>"/><br/>
    <textarea name="body"><%=article.body%></textarea><br/>
    <input type="submit" value="Update Article"/>
</form>
```

**NOTE: to compare ObjectIds, which are objects, you must first convert them to Strings (e.g. articleAuthor._id.toString())**

Update the PUT route in controllers/articles.js

```javascript
router.put('/:id', (req, res)=>{
    Article.findByIdAndUpdate(req.params.id, req.body, { new: true }, (err, updatedArticle)=>{
        Author.findOne({ 'articles._id' : req.params.id }, (err, foundAuthor)=>{
		if(foundAuthor._id.toString() !== req.body.authorId){
			foundAuthor.articles.remove(req.params.id);
			foundAuthor.save((err, savedFoundAuthor)=>{
			
				Author.findById(req.body.authorId, (err, newAuthor)=>{
					newAuthor.articles.push(updatedArticle);
					newAuthor.save((err, savedNewAuthor)=>{
			        	        res.redirect('/articles/'+req.params.id);
					});
				});
			});
		} else {
			res.redirect('/articles/' + req.params.id)
		}
        });
    });
});
```
