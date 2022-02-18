# API Builder SOAP To REST Demo

Simple API Builder demo to show how to convert a SOAP service to a REST service.

The SOAP Service returns the capitol city for a given country code. The SOAP service is called as follows:

```
curl --location --request POST 'http://webservices.oorsprong.org/websamples.countryinfo/CountryInfoService.wso' \
--header 'Content-Type: text/xml; charset=utf-8' \
--data-raw '<?xml version="1.0" encoding="utf-8"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <CapitalCity xmlns="http://www.oorsprong.org/websamples.countryinfo">
      <sCountryISOCode>US</sCountryISOCode>
    </CapitalCity>
  </soap:Body>
</soap:Envelope>'
```

Sample test cases are shown below:

* US - Washington
* BR - Brasilia
* DE - Berlin
* CA - Ottowa

The API Builder REST API is created by modifying the default Greeting API which is a sample REST API that is provided out of the box when you create a new API Builder project. This is done for simplicity sake. In practice you would define your API in an API design tool such as [Stoplight](https://stoplight.io/) and import it into to API Builder in order to build the flow that implements the SOAP to REST conversion.

The modified Greeting API is shown below:

![](https://i.imgur.com/IDoRoJv.png)

You can see it in action below:

![](https://i.imgur.com/40OO1HG.png)

Since we are using the Greeting API, we will use the username query parameter for our country code.

It uses the following flow nodes to accomplish the SOAP to REST conversion:
* [**XML**](https://github.com/Axway-API-Builder-Ext/api-builder-extras/tree/master/api-builder-plugin-fn-xml-node) community plugin to convert the SOAP XML response to JSON
* Mustache - for creating the SOAP service input XML based on the REST query parameter
* REST - to call the SOAP service
* Stringify - to extract the desired response from the SOAP envelope response
* Parse - to parse the JSON string output from the Stringify flow node
