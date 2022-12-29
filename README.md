# SOAP is dead - long live SOAP

First of all do not write SOAP services if you can avoid it! It is over.

If you can not avoid it this package might help.

## Service

```go
package main

import (
	"encoding/xml"
	"fmt"
	"net/http"

	"github.com/foomo/soap"
)

// FooRequest a simple request
type FooRequest struct {
	XMLName xml.Name `xml:"fooRequest"`
	Foo     string
}

//Example for a header content 
type SecurityRequest struct {
	XMLName xml.Name `xml:"Security"`
	Login   UserNameToken `xml:"MyToken"`
}

type MyToken struct {
	DataToken string `xml:"datatoken"`
}

// FooResponse a simple response
type FooResponse struct {
	Bar string
}

// RunServer run a little demo server
func RunServer() {
	soapServer := soap.NewServer()
	soapServer.HandleOperation(
		// SOAPAction
		"operationFoo",
		// tagname of soap body content
		"fooRequest",
		// HeaderFactoryFunc - give the server sth. to unmarshal the request header into
		func() interface{} {
			return &SecurityRequest{}
		},
		// RequestFactoryFunc - give the server sth. to unmarshal the request content into
		func() interface{} {
			return &FooRequest{}
		},
		// OperationHandlerFunc - do something
		func(header interface{}, request interface{}, w http.ResponseWriter, httpRequest *http.Request) (response interface{}, err error) {
			fooRequest := request.(*FooRequest)
			headerRequest := header.(*SecurityRequest)

			fmt.Println("Info header: ", headerRequest.OtherData) 

			fooResponse := &FooResponse{
				Bar: "Hello " + fooRequest.Foo,
			}
			response = fooResponse
			return
		},
	)
	err := soapServer.ListenAndServe(":8080")
	fmt.Println("exiting with error", err)
}

func main() {
	RunServer()
}
```

## Client

```go
package main

import (
	"encoding/xml"
	"log"

	"github.com/tianpl92/soap"
)

// FooRequest a simple request
type FooRequest struct {
	XMLName xml.Name `xml:"fooRequest"`
	Foo     string
}

// FooResponse a simple response
type FooResponse struct {
	Bar string
}

func main() {
	soap.Verbose = true
	client := soap.NewClient("http://127.0.0.1:8080/", nil, nil)
	response := &FooResponse{}
	httpResponse, err := client.Call("operationFoo", &FooRequest{Foo: "I am foo"}, response)
	if err != nil {
		panic(err)
	}
	log.Println(response.Bar, httpResponse.Status)
}

```

## With Api tester like POSTMAN


```xml

<?xml version="1.0" encoding="utf-8"?>
<soapenv:Envelope
    xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
    <soapenv:Header
        xmlns:wsa="http://schemas.xmlsoap.org/ws/2004/08/addressing"
        xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd"
        xmlns:wsse="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd">
        <wsse:Security>
            <wsse:MyToken
                xmlns:wssu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd"
                xmlns="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd">
                <wsse:datatoken>OTHER INFO</wsse:datatoken>
            </wsse:MyToken>
        </wsse:Security>
    </soapenv:Header>
    <soapenv:Body>
        <fooRequest>
          <Foo>I am foo</Foo>
    	</fooRequest>
    </soapenv:Body>
</soapenv:Envelope>

```