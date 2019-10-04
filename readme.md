# Exploring Erlang setup

## Erlang and the new space


## In depth

* [Stack](documentation/stack.md)
* [Deployment](documentation/deployment.md)
* [Development](documentation/development.md)
* [Monitoring](documentation/monitoring.md)



## API

- / - root
- /healthy - returning header **X-Health:Awsome** - used for liveness http get probe
- /:id - returning :id
- /bgg/item/:id - TODO: make this one load item from BGG by id
- /bgg/hot - returns json with hottest boardgames list from bGG