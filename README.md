# Trolly

A wrapper around the Trello API. Provides a group of classes to represent Trello Objects. None of the classes cache 
values as they are designed to be inherited and extended to suit the needs of each user. Each class includes a basic 
set of methods based on general use cases. This library was based on work done by 
[sarumont](https://github.com/sarumont/py-trello). Very little was kept from this code, but still props on the initial 
work.


## Getting Started

### Dependencies

This library requires python 2.5 and above.

Before getting stated with this library you will need a few extra things:
- [httplib2](http://code.google.com/p/httplib2/)
- An [API key](https://trello.com/docs/gettingstarted/index.html#getting-an-application-key) for your Trello user
- User authorisation token ( see below for how to obtain )

### Authorisation

#### User Authorisation Token

A user authorisation token isn't too hard to get hold of. There are instruction on how to get one on the 
[Trello](https://trello.com/docs/gettingstarted/index.html#getting-a-token-from-a-user). For those too lazy there is a 
python class in the library called Authorise(). To use this class simply navigate to the file in a terminal and type:
    
    python authorise.py -a API_KEY APPLICATION_NAME WHEN_TO_EXPIRE

The API key and application names are required but the "WHEN_TO_EXPIRE" will default to 1day if not specified. Running
this file will return a URL. Copy and paste it into your browser and away you go. You might want to store this somewhere
for future use, especially if you have set it to never expire.

#### Oauth

This library (currently) has no Oauth support however the code this was based on includes Oauth support. So for 
inspiration on how to extend the Client class to include this check out the link above.


## Overview

### Trello Client

This class holds the bulk of all the methods for communicating with the trello API and returning the Trello objects. 
A client instance is required by every Trello object, becaause of this it makes extending and overiding methods in this
class very effective as it effects all objects simultaneously.


### Trello Object

This class is inhereted my all Trello object classes ( Board, List, Card, etc ). The class takes only one argument, a 
Trello client instance. It also masks calls to the client as belonging to the class ( for creating objects ). This 
allows the library to be more modular.


### Extending Trello Classes

Extending these classes is the premise on which they were built. Below outlines an example of how this can be acheived.

If for example we wanted to pass extra variables to our a Trello Card object then we can do the below:

    class MyList( List ):

        def __init__( self, trello_client, list_id, name = '' ):
            super( MyList, self ).__init__( trello_client, list_id, name )

        def getCards( self ):
            cards = self.fetchJson( uri_path = self.base_uri + '/cards' )
            return self.createCard( card_json = cards[0], test = 'this is a test argument' )


This class overrides the getCards method to add the extra variable we need. This will need to be done to any Trello object that will return a custom card or you can use:

    kwargs.get('test',"default value")

This will help avoid a value not being passed. You could also instead of extending the object creation you could add
a method to cache the details you want from all the object getObjectInformation method.


We declare and pass the extra ('test') variable as a keyword argument here. 
We then need to extend the card class to allow for the extra variables:

    class MyCard( Card ):

        def __init__( self, trello_client, card_id, test, name = '' ):
            super( MyCard, self ).__init__( trello_client, card_id, name )
            self.test_arg = test

Finally, we extend and override the Client. Overriding the client means that any object that calls createCard will 
create one of our new client classes.

    class MyClient( Client ):

        def __init__( self, api_key, user_auth_token ):
            super( MyClient, self ).__init__( api_key, user_auth_token )

        def createCard( self, card_json, **kwargs ):

            return MyCard( 
                    trello_client = self,
                    card_id = card_json['id'],
                    name = card_json['name'],
                    test = kwargs['test']
                )

Hope this helps!

## Licence

This code is licenced under the [MIT Licence](http://opensource.org/licenses/mit-license.php)