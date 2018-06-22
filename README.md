# remedy-rest-api

This is a javascript library for talking to [BMC's Remedy ARS REST API](https://docs.bmc.com/docs/ars91/en/bmc-remedy-ar-system-rest-api-overview-609071509.html).

The goal is to make a zero-dependency library that can work either in node.js or in a browser. This version *mostly* acheives that goal. The library *is* zero-depdendency within a browser. To use it with node.js, we do need one dependency to emulate the XMLHttpRequest API.

at some point in the future, I'm sure I'll hook this up to something slick, but at the moment if you want to use it in a browser ... well ... *probably it'll work* if you comment out lines 38, 39 and the very last line (module.exports ...). As of this release,
I haven't tested this in a browser. In theory it should work, as I've not used any unsupported aspects of the XHR API, but
anyhow, at some point I'll circle back and work on that part. Or feel free to contribute! :-)

this is an object oriented library. All of the functions that talk to the AR Server are asynchronous and return a Promise.

the test.js does a pretty good job of showing how everything works. Here's a quick overview:

```javascript

// include the library (node.js)
const Remedy = require('remedy-rest-api');

// create an object (does not return a promise)
try {
    let api = new Remedy({
        protocol:       'https',
        server:         'fakeserver.hicox.com',
        user:           'fakeAccount',
        pass:           'fakePass',
        timeout:        20000
    });
}catch(e){
    // do something with the error
}

// log into the ARServer (returns promise)
api.authenticate().then(function(self){
    // self contains a reference to api
}).catch(function(e)){
    // something failed, hannle yo bidness
}


/*
    you can also instantiate a new object
    and chain it to authenticate all in one
    go, and if you do it all inside an
    async function you can use await which
    will make you smile ...
*/
async function doSomeRemedyStuff(){

    // log into remedy and get ready to do stuff
    let api = await new Remedy({
        protocol:       'https',
        server:         'fakeserver.hicox.com',
        user:           'fakeAccount',
        pass:           'fakePass',
        timeout:        20000
    }).authenticate().catch(function(e){
        /*
            all errors returned by the library are of the ARSRestException class
            there are a lot of nifty features and I'll write more about this later
            however, the *.toString attribute is quite handy ...
        */
        throw(`failed login!: ${e.toString}`);
    });



    // create a ticket
    let newTicket = await api.createTicket({
        schema:     <formName>,
        fields:     {
            fieldNameOne:   "valueNumber1",
            fieldNameTwo:   "valueNumber2"
            ... etc ...
        }
    }).catch(function(e){
        throw(`failed createTicket! ${e.toString}`);
    });
    console.log(`I made a new ticket! ${newTicket.entryId}`);



    // modify a ticket
    await api.modifyTicket({
        schema:     <formName>
        ticket:     <entryId>
        fields:     {
            fieldNameOne:   "a new value"
        }
    }).catch(function(e){
        throw(`failed modifyTicket: ${e.toString}`)
    });



    // retrieve field values from a ticket when you know the ticket number
    let myTicketData = await api.getTicket({
        schema:     <formName>
        ticket:     <entryId>
        fields:     ['List', 'Of', 'FieldNames', 'to get', 'values for']
    }).catch(function(e){
        throw(`failed getTicket: ${e.toString}`)
    });
    console.log("Here's what I got!")
    Object.keys(myTicketData.values).forEach(function(fieldName){
        console.log(`\t[${fieldName}]: ${myTicketData.values[fieldName]}`);
    });



    // query for tickets
    let myRecordList = await api.query({
        schema:     <formName>
        QBE:        `'Entry ID' != $NULL$`
        Fields:     ['List', 'Of', 'FieldNames', 'to get', 'values for']
    }).catch(function(e){
        throw(`query failed: ${e.toString}`);
    });
    console.log("here's all the records I got!");
    myRecordList.entries.forEach(function(row, rowNumber){
        console.log(`[row number]: ${rowNumber}`);
        Object.keys(row.values).forEach(function(fieldName){
            console.log(`\t[${fieldName}]: ${row.values[fieldName]}`);
        });
    })


    // delete a ticket
    let theDeletedTicketNumber = await api.deleteTicket({
        schema:     <formName>
        ticket:     <entryId>
    }).catch(function(e){
        throw(`failed to delete ticket: ${e.toString}`);
    })
    console.log(`I just deleted: ${theDeletedTicketNumber}`);


    /*
        merge works!
        there are a lot of options.
        I'll write better documentation later
        test.js has a fairly readable test of everything
        might wanna check that out in the mean time
    */


    /*
        create a record on the given form with the given data
        if the 'Entry ID' (field #1 on the form) is contained
        in the given data, and there is already a record with
        that 'Entry ID', throw an error
    */
    let ticketId = await api.mergeData({
        schema:                 <ticket>,
        handleDuplicateEntryId: "error",
        fields: {
            'Entry ID':             'BOGUS-000000001',
            'Short Description':    'cowabunga dudes!'
            ...
        }    
    }).catch(function(e){
        throw(`mergeData failed: ${e.toString}`);
    });
    console.log(`merged data onto ${ticketId.entryId}`);

}


```

yeah ok, I'm just out of time for now.
Will write more docs later
