# notes


## React Native



### Native recycling list

I know there are some examples about this native list view, and I looked into it, it won't work with in production app.

- Currently solution suggestion use internal reactjs method to access node, and reactjs changed
- Even above works, current solution suggest to bind data in plain array, usually this won't work with RealmDB.

Simple explaination of current solution:

- To create a native view, by using children props in react native
- To establish a queue for recycle cells by it
- To create a UITableView in this view
- To provide cell from the above queue to native threading-pool
- To update the cell presentational component by UIManager.updateView in native code
- To update the JS based component inside native, it requires { tag, viewType, props }
- To provide the {tag, viewType, props} with data-source by Reactjs internal method in JS

By upgrading the reactjs internal method of latest version, the example working well. 

But if you consider to use a data source like realmDB list, then you can't do it.

As its not a plain array or objects, to transform them into plain array or object, will be cost.

### RealmDB

RealmDB is easy to use with good performance.

How it works?

In react native, realm-js will try to find current context, if it's in react-native or javascript core, and then it will try to get the realm access handle.

The way to access realm at native side, they provide two different scenarios:

- To access the db at debug mode, as long as it's debug mode, the javascript is running inside desktop chrome javascript engine, and then every native access, will be delegate to the real app.

The realm provide a web server at mobile or simulator, and then by using Ajax to send command to the server when its in debug mode, this process is sync one.

- To access the db at production mode, at this time, the Realm will inject a Object to javascript core, just like the ```window``` object in javascript, then we can access the Realm functionality by using this Object directly, this process is also sync.

The real question is that, we can see, the way realm-js to access db, is a lot different from react native native module method, and it do provide a pretty concise way to support all different js environment, including nodejs.

But it also bring a small issue in react native, as long as it using Realm global object in JS, it also provide a special type of list to query result from realm db.

This list is special proxy, it only provide the real data from db, as soon as the data is read by JS.

This lazy loading sounds very good, in some case, it's not really good.

For example, if we just load 1000+ data into the list, and then put them into the list view, it will be really slow to render them same time in javascript.

Also, to do any statistics on them, sum all value, or just calculate the average value, will be very slow, because they are not in memory yet, and even then, this operation will be running in a sync threading of javascript core, so the UI will frozen, and we can do nothing about it.

A better way?

A better way to do large data set processing, is to put them on another thread beside javascript core and main thread of the app. This is how react native module do, so it should provide a way to access the realm db data in native module.

Yes, it do.

But, to access the realm db at native module in java, it required to write the schema model at java side as well, so this time, will have to write two copy of the schema in javascript or java.

As long as RealmResult<T> is defined, it will require type like T, in java, so its not avoidable.

I just found a way:

> https://github.com/realm/realm-java/tree/master/examples/migrationExample

> https://realm.io/docs/java/latest/#migrations

In migration, it do have a way to get schema.

This is no the solution.

The real solution in java is to use Dynamic Realm, but its not in objc API yet.

