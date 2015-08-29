React is a very powerful library for building interactive JavaScript
applications.  One of its strongest qualities is that it forces you to build 
your app in a uni-directional data flow.  This is much different from other 
frameworks’ two-way data binding. With[React][1], every piece of your
application is controlled from a single source, it’s owner.  The owner is 
defined by the Component that is passing “props” to its owned component.

As more nested components are added, this logic can grow out of hand and can
make it difficult to keep track of events passed through the component hierarchy.
 This is where[Flux][2] steps in to provide an outlet from the hierarchy to
handle dispatched events and altered data within our application.

Rather than going into graphs and models to describe what Flux does, lets build
an easy Flux example application.  We’ll be building our application with the 
help of ES6 and Babel.  If you’re unfamiliar with these, you can
[find out more about ES6 and Babel here][3]. Babel allows us to use both ES6
JavaScript and JSX which will both compile to ES5 JavaScript which is compatible
across all modern and legacy browsers. Now let’s get started
…

###### TL;DR

[Click here to download the full source code for this tutorial on GitHub][4].
Or keep reading if you want to follow along step-by-step.

#### Getting started

Get started by running the following commands in your preferred location:

    mkdir easy-flux-example
    cd easy-flux-example
    

Now we need to add our dependencies. Normally you may just get a package.json
file and hit npm install, but it’s good for educational purposes to know exactly
what you’re adding to your project and why:

    npm install react // for react
    npm install babelify // babel compiler
    npm install browserify // node module loader
    npm install flux // adds flux into the mix
    npm install gulp // our taskrunner to run compile / build processes
    npm install lodash // a great JS utility
    npm install vinyl-source-stream // helps source our files for easy gulping
    npm install events // node's event emitter
    

Now that we’ve stocked our toolkit, let’s build out our task runner. My
preference is[Gulp][5], but you could just as easily use [Grunt][6] (though
there are[legions of developers who would tell you Gulp is the better way][7],
myself included
).

    vim gulpfile.js
    

And add the following:

    // gulpfile.js
    var gulp = require('gulp');
    var browserify = require('browserify');
    var babelify = require('babelify');
    var source = require('vinyl-source-stream');
     
    gulp.task('build', function () {
      browserify({
        entries: 'app.js',
        extensions: ['.jsx'],
        debug: true
      })
      .transform(babelify)
      .bundle()
      .pipe(source('bundle.js'))
      .pipe(gulp.dest('dist'));
    });
     
    gulp.task('default', ['build']);
    

So looking at this file you can see that the default build will use Browserify
to grab all of our required modules and pipe them to Babel to be transformed 
into a bundle.js file and will end up in a folder called “dist”. Pretty simple 
and straight-forward.

Now that we have all of our processes ready to go, let’s build our
application (finally!
).

Let’s start by creating index.html

    vim index.html
    

Add the following to index.html:

    <!DOCTYPE html>
    <html lang="en">
    <head>
    	<meta charset="UTF-8">
    	<title>Easy Flux Example</title>
    </head>
    <body>
    	<h1>Easy Flux Example</h1>
    	<div id="app-root"></div>
    	<script src="dist/bundle.js"></script>
    </body>
    </html>

And that’s all we need for our frontend, everything else will be handled by
React and Flux.

Let’s get started with our application logic by adding our main app file.

    vim app.js
    

Add the following to app.js

    // app.js
    import React from 'react';
    import AppRoot from './components/AppRoot'; // we'll create this next
    
    React.render(<AppRoot />,
      document.getElementById('app-root')
    );
    
    

Notice all of the ES6 syntax which Babel will sort out into cross-compatible
ES5 JavaScript. Also notice the importing of node modules, which Browserify will
handle.

Next, let’s start building our Components, starting with AppRoot.

    mkdir components
    cd components
    vim AppRoot.jsx
    

And add the following:

    // AppRoot.jsx
    import React from 'react';
    class AppRoot extends React.Component {
        render(){
            let itemHtml = <li>Hello React</li>; // we will map a list here later
            return <div>
                <ul>
                    { itemHtml }
                </ul>
             </div>;
        }
      };
    
    export default AppRoot;
    

Now let’s take a step back and make sure all of this is working properly.

    cd ../

Run our gulp command which will convert our ES6 and JSX into ES5 (Babelify) and
load everything into bundle.js (Browserify
):

    gulp

Let’s check out what we have so far. You could easily just view index.html in
your browser, but I like to view my apps on localhost. If you don’t have http-
server installed, run the following:

    sudo npm install http-server -g
    http-server

Now go to <http://localhost:8080>. You should see a message which means that
your React application is working.

Now that React is working, let’s add some interactivity, which is where Flux
will work its magic.

#### Adding Flux

The main pieces of the Flux architecture are the Components, Store and
Dispatcher. Since we have our first Component set up, lets get started with our 
Store. Our Store will satisfy three pieces of our application. It will:

1. Get initial data for our application.  
2. Store the functions for our application processes.  
3. Emit events that will tell our application to re-render our application.

#### Reading data from the Store

To get our Store set up run the following commands:

    mkdir stores
    cd stores
    vim ListStore.js
    

In ListStore.js add the following:

    // ListStore.js
    import {EventEmitter} from 'events';
    import _ from 'lodash';
    
    let ListStore = _.extend({}, EventEmitter.prototype, {
    
      // Mock default data
      items: [
        {
          name: 'Item 1',
          id: 0
        },
        {
          name: 'Item 2',
          id: 1
        }
      ],
    
      getItems: () => {
        return this.items;
      },
    
    });
    
    export default ListStore;

Here, we’ve added Node’s EventEmmiter so our Store can emit events to
trigger our UI changes. Lodash helps use bind the EventEmmiter to our ListStore 
object (you could also use other methods for this). And our mock data will be 
the default data visible on render.`getItems` is the obvious function that will
get items from our Store.

Next, let’s add the new ListStore to our AppRoot Component so we can render
our default Store data. Run the following commands:

    cd ../
    cd components
    vim AppRoot.jsx
    

Replace the contents of AppRoot.jsx with the following:

    // AppRoot.jsx
    import React from 'react';
    import ListStore from '../stores/ListStore';
    
    // Method to retrieve state from Stores
    function getListState() {
      return {
        items: ListStore.getItems()
      };
    }
    
    class AppRoot extends React.Component {
    
      constructor() {
        super();
        this.state = getListState();
      }
    
      render(){
          
        let items = ListStore.getItems();
        
        let itemHtml = items.map(( listItem ) => {
    
          return <li key={ listItem.id }>
              { listItem.name }
            </li>;
    
        });
    
        return <div>
            <ul>
                { itemHtml }
            </ul>
    
        </div>;
      }
    
    }
    
    export default AppRoot;
    

Now run the following commands to rebuild your bundle.js:

    cd ../
    gulp
    

Now run the following command:

    http-server
    

And check out the changes at <http://localhost:8080>

#### Adding / removing items in the Store

Now let’s add the ability to write new items to our Store. Let’s create a
new component that will handle this new functionality. Run the following 
commands:

    cd components
    vim NewItemForm.jsx
    

And now add the following to NewItemForm.jsx:

    // NewItemForm.jsx
    import React from 'react';
    import ListStore from '../stores/ListStore';
    import AppDispatcher from '../dispatcher/AppDispatcher';
    
    class NewItemForm extends React.Component {
    
      createItem(e){
        
        // so we don't reload the page
        e.preventDefault();
        // get all items
        let items = ListStore.getItems();
        // this will always be 1 more than the last item index
        let id = items.length;
        
        // this gets the value from the input
        let item_title = React.findDOMNode(this.refs.item_title).value.trim();
        
        // this removes the value from the input
        React.findDOMNode(this.refs.item_title).value = '';
        
        // This is where the magic happens, 
        // no need to shoot this action all the way to the root of your application to edit state.
        // AppDispatcher does this for you.
        AppDispatcher.dispatch({
          action: 'add-item',
          new_item: {
            id: id,
            name: item_title
          }
        });
    
      }
    
      render(){
    
        return <form onSubmit={ this.createItem.bind(this) }>
            <input type="text" ref="item_title"/>
            <button>Add new item</button>
          </form>
      }
    }
    
    export default NewItemForm;
    

So to give you a brief idea of what’s happening here, from the render up: We
’re showing a text input that has the function`createItem` bound to it on
submit. When the form is submitted,`createItem` will get the length of items in
ListStore then create the new id, clear the input, then pass the action (‘add-
item’) and the`new_item` data to the AppDispatcher.

Now we need to create our AppDispatcher to receive this data. Run the following
commands:

    cd ../
    mkdir dispatcher
    cd dispatcher
    vim AppDispatcher.js
    

Add the following to AppDispatcher.js:

    import {Dispatcher} from 'flux';
    let AppDispatcher = new Dispatcher();
    
    import ListStore from '../stores/ListStore';
    
    // Register callback with AppDispatcher
    AppDispatcher.register((payload) => {
    
      let action = payload.action;
      let new_item = payload.new_item;
      let id = payload.id;
    
      switch(action) {
    
        // Respond to add-item action
        case 'add-item':
          ListStore.addItem(new_item);
          break;
        
        // Respond to remove-item action
        case 'remove-item':
          ListStore.removeItem(id);
          break;
    
        default:
          return true;
      }
    
      // If action was responded to, emit change event
      ListStore.emitChange();
    
      return true;
    
    });
    
    export default AppDispatcher;
    

This is where Flux can get confusing, but with this simple example, I think it
’s pretty clear that when AppDispatcher is called we can perform our logic on 
its payload to determine what actions to take and what data to transmit. If the 
action is ‘add-item
’ `ListStore.addItem` is triggered. If the action is ‘remove-item’ then 
`ListStore.removeItem` is triggered. Since these two functions don’t exist in
our Store yet, let’s add them to our ListStore.

Run the following commands:

    cd ../stores
    vim ListStore.js
    

Replace the contents of ListStore.js with the following:

    // ListStore.js
    import {EventEmitter} from 'events';
    import _ from 'lodash';
    
    let ListStore = _.extend({}, EventEmitter.prototype, {
    
      // Mock default data
      items: [
        {
          name: 'Item 1',
          id: 0
        },
        {
          name: 'Item 2',
          id: 1
        }
      ],
    
      // Get all items
      getItems: () => {
        return this.items;
      },
    
      // Add item
      addItem: function(new_item){
        this.items.push(new_item);
      },
    
      // Remove item
      removeItem: (item_id) => {
        
        let items = this.items;
        
        _.remove(items,(item) => {
          return item_id == item.id;
        });
        
        this.items = items;
    
      },
    
      // Emit Change event
      emitChange: () => {
        this.emit('change');
      },
    
      // Add change listener
      addChangeListener: (callback) => {
        this.on('change', callback);
      },
    
      // Remove change listener
      removeChangeListener: (callback) => {
        this.removeListener('change', callback);
      }
    
    });
    
    export default ListStore;
    

Notice what we’ve added here: functions `addItem` and `deleteItem` add or
remove items to`this.items`. `emitChange` causes an event to be emitted to our
AppRoot Component with`_onChange` which will then render the new state from the
ListStore.

Now we need to edit our AppRoot Component to listen for the new events being
triggered by the ListStore.

Run the following commands:

    cd ../components
    vim AppRoot.jsx
    

And now we’ll replace the contents of AppRoot with the following:

    // AppRoot.jsx
    import React from 'react';
    import ListStore from '../stores/ListStore';
    import AppDispatcher from '../dispatcher/AppDispatcher';
    
    // Sub components
    import NewItemForm from './NewItemForm';
    
    // Method to retrieve state from Stores
    let getListState = () => {
      return {
        items: ListStore.getItems()
      };
    }
    
    class AppRoot extends React.Component {
      
      // Method to setState based upon Store changes
      _onChange() {
        this.setState(getListState());
      }
    
      constructor() {
        super();
        this.state = getListState();
      }
    
      // Add change listeners to stores
      componentDidMount() {
        ListStore.addChangeListener(this._onChange.bind(this));
      }
    
      // Remove change listeners from stores
      componentWillUnmount() {
        ListStore.removeChangeListener(this._onChange.bind(this));
      }
    
      removeItem(e){
    
        let id = e.target.dataset.id;
        
        AppDispatcher.dispatch({
          action: 'remove-item',
          id: id
        });
    
      }
    
      render(){
          
        let _this = this;
        let items = ListStore.getItems();
        let itemHtml = items.map(( listItem ) => {
          return <li key={ listItem.id }>
              { listItem.name } <button onClick={ _this.removeItem } data-id={ listItem.id }>×</button>
            </li>;
        });
    
        return <div>
            <ul>
                { itemHtml }
            </ul>
            <NewItemForm />
    
        </div>;
      }
    
    }
    
    export default AppRoot;
    

Notice what we’ve added here: `_onChange` handles all state changes within our
Flux app. And notice that we have the function`removeItem` which calls the
AppDispatcher with the “remove-item” action.

Now let’s see if it’s working as it should. Run the following commands to
build the bundle.js with our updates.

    cd ../
    gulp
    

And when gulp is done, run:

    http-server
    

Now go to <http://localhost:8080> and you should be able to add and remove
items from our easy Flux example application.

I hope you found this tutorial helpful in understanding the powerful concepts
behind building interactive apps with React and Flux. If you would like to 
download the whole thing, you can
[download the easy Flux example application here][4].

If you have any questions, comments, suggestions, feel free to leave them in
the comments section below or[reach out to me on twitter][8].

– Tony Spiro

 [1]: http://facebook.github.io/react/
 [2]: https://facebook.github.io/flux/
 [3]: https://babeljs.io/
 [4]: https://github.com/tonyspiro/easy-flux-example
 [5]: http://gulpjs.com/
 [6]: http://gruntjs.com/
 [7]: http://sixrevisions.com/web-development/grunt-vs-gulp/
 [8]: http://twitter.com/tonyspiro