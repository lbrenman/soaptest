# API Builder SOAP To REST Demo

Simple API Builder demo to show how to convert a SOAP service to a REST service.

The CountryInfoService SOAP Service returns the capitol city for a given country code. The SOAP service is called as follows:

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

And has the following response:

```
<?xml version="1.0" encoding="utf-8"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <m:CapitalCityResponse xmlns:m="http://www.oorsprong.org/websamples.countryinfo">
      <m:CapitalCityResult>Washington</m:CapitalCityResult>
    </m:CapitalCityResponse>
  </soap:Body>
</soap:Envelope>
```

Sample test cases are shown below:

* US - Washington
* BR - Brasilia
* DE - Berlin
* CA - Ottowa
* IL - Jerusalem

Since API Builder APIs are always REST API's all we need to do is create the XML body from the REST  parameter(s) and convert the SOAP XML response to a REST/JSON reply.

For this demo, the API Builder REST API is created by modifying the default Greeting API which is a sample REST API, provided out of the box when you create a new API Builder project. This is done for simplicity sake. In practice you would define your API in an API design tool such as [Stoplight](https://stoplight.io/) and import it into to API Builder in order to build the flow that implements the SOAP to REST conversion.

The modified Greeting API is shown below:

![](https://i.imgur.com/YFgrn4V.png)

You can see it in action below:

![](https://i.imgur.com/40OO1HG.png)

Since we are using the Greeting API, we will use the username query parameter for our country code.

Before calling the CountryInfoService SOAP service using the REST flow node, we need to construct the SOAP body using a Mustache flow node as follows. The Data value is set to `$.params` as a Selector. The Mustache template is show below:

```
<?xml version="1.0" encoding="utf-8"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <CapitalCity xmlns="http://www.oorsprong.org/websamples.countryinfo">
      <sCountryISOCode>{{data.username}}</sCountryISOCode>
    </CapitalCity>
  </soap:Body>
</soap:Envelope>
```

The Mustache flow node output is set to `$.soapMessage.value`.

The call to the CountryInfoService SOAP service is performed using the REST flow node with the following Parameters:

* Body - Selector, `$.soapMessage.value`
* URL - String, `http://webservices.oorsprong.org/websamples.countryinfo/CountryInfoService.wso`
* Headers - Object, `{"Content-Type": "text/xml; charset=utf-8"}`

The output is unchanged. `$.response` for the 2XX, 3XX, 4XX and 5XX outputs and `$.error` for the error output.

After calling the CountryInfoService SOAP service, we convert the XML response, `$.response`, to JSON using the  [**XML**](https://github.com/Axway-API-Builder-Ext/api-builder-extras/tree/master/api-builder-plugin-fn-xml-node) community plugin.

The response looks like this:

```
<?xml version="1.0" encoding="utf-8"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <m:CapitalCityResponse xmlns:m="http://www.oorsprong.org/websamples.countryinfo">
      <m:CapitalCityResult>Washington</m:CapitalCityResult>
    </m:CapitalCityResponse>
  </soap:Body>
</soap:Envelope>
```

We set the XML flow node "XML Input data" to `$.response.body` and set the output to `$.jsonData`.

Then we assemble HTTP response using the HTTP flow node and set the Body to: `$.jsonData["soap:Envelope"]["soap:Body"]["m:CapitalCityResponse"]["m:CapitalCityResult"]` in order to simply get the capitol city name.

This makes the resulting REST API response (for username=DE) look like the following:

```
{
	"status": 200,
	"body": "Berlin"
}
```
