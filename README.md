# TD Ameritrade Java Client
Java rest client for TD Ameritrade Api. Uses [OKHttp 3](https://github.com/square/okhttp) under the hood.

I'm happy to collaborate contractually or OSS with other devs. Some of the methods are hard to test simply because
I do not wish to make actual trades during business hours and hope that they can be cancelled in time and with working code.
Unfortunately, I have not found a way to use one of TDA's *ThinkOrSwim* simulation accounts
with this API as TDA does not provide that route. 

## 2018 Notes

Sometime in-between the beginning of this project (based on TDA's older XML API) and now, TDA released a restful [API](https://developer.tdameritrade.com/). 
TDA's server backend is still similar, as far as I can tell. IOW, from the various 
APIs TDA has released over the years, I estimate that their internal backend is accessed using old-school XML 
auto-generated methods and POJOs in Java. Both the old XML API which this project uses and the new rest API have almost
identical return data, the only difference being one is XML and the other JSON (generated from the XML).  

Also, this API requires only the user name and password (same as the site itself) while the new API uses OAuth.

## Build

To build the jar, checkout the source and run:

```bash
mvn clean install
```

## Usage
Until the project is finished, you will need to have built this locally in order to put the necessary jars in your local Maven repo.
Once we have a 1.0.0 version, it will be submitted to Maven Central. 

Add the following to your Maven build file:

```
  <dependency>
    <groupId>com.studerw</groupId>
    <artifactId>tdameritrade-api</artifactId>
    <version>1.0-SNAPSHOT</version>
  </dependency>
```

You need a valid TDA user and password which is the same that is used when logging into the [TDA Home Page](https://www.tdameritrade.com/home.page)

```
  TdaClient tdaClient = new HttpTdaClient("user", "pass".getBytes());
  QuoteResponse response = tdaClient.fetchQuotes(Arrays.asList("AMTD"));
  String askPrice = response.getResults.getQuotes().get(0).getAsk();
  System.out.println("Current price of TDA: " + askPrice);
```

## Response Types
The actual responses returned from TDA are in rather verbose XML. For example, a call to get several ticker quotes
will return something like this:

```Java
QuoteResponse response = httpTdaClient.fetchQuotes(Arrays.asList("msft", "xom"));
System.out.println(response.getOriginalXml());
```

```xml
<?xml version="1.0"?>
<amtd>
    <result>OK</result>
    <error></error>
    <quote-list>
        <error></error>
        <quote>
            <error></error>
            <symbol>MSFT</symbol>
            <description>Microsoft Corporation - Common Stock</description>
            <bid>93.52</bid>
            <ask>93.64</ask>
            <bid-ask-size>1100X100</bid-ask-size>
            <last>93.64</last>
            <last-trade-size>47600</last-trade-size>
            <last-trade-date>2018-03-05 16:00:02 EST</last-trade-date>
            <open>92.34</open>
            <high>94.27</high>
            <low>92.26</low>
            <close>93.05</close>
            <volume>23901578</volume>
            <year-high>96.07</year-high>
            <year-low>63.62</year-low>
            <real-time>true</real-time>
            <exchange>NASDAQ</exchange>
            <asset-type>E</asset-type>
            <change>0.59</change>
            <change-percent>0.63%</change-percent>
        </quote>
        <quote>
            <error></error>
            <symbol>XOM</symbol>
            <description>Exxon Mobil Corporation Common Stock</description>
            <bid>76.29</bid>
            <ask>76.93</ask>
            <bid-ask-size>1500X200</bid-ask-size>
            <last>76.27</last>
            <last-trade-size>2384300</last-trade-size>
            <last-trade-date>2018-03-05 16:00:49 EST</last-trade-date>
            <open>75.23</open>
            <high>76.56</high>
            <low>75.121</low>
            <close>75.55</close>
            <volume>14890975</volume>
            <year-high>89.30</year-high>
            <year-low>73.90</year-low>
            <real-time>true</real-time>
            <exchange>NYSE</exchange>
            <asset-type>E</asset-type>
            <change>0.72</change>
            <change-percent>0.95%</change-percent>
        </quote>
    </quote-list>
</amtd>
```

In java, you will get a `QuoteResponse` pojo. All of the resonse objects extend the base `BaseTda`
which has convenience methods:

* `getOriginalXml` - get the XML as above, though shouldn't be needed except for special purposes.
* `isTdaError` - did the response return a processing error.

Getting the data out of the response objects is a bit complex because of the generated types. Above, to
get the `askPrice` for Exxon, it looks like this:

```java
response.getResult().getQuotes().get(1).getAsk();
```

## Error Handling

The TDA server returns 200 success responses **even if the call was bad**, for example you cannot login
or some other issue. Instead of a bad response, their is always an `error` field filled in with something
like `OK` for successful calls and `error` or otherwise, along with possibly an error message, when something
has gone wrong. 

The rules are this within the Client:
* All non 200 HTTP responses throw unchecked `RuntimeExceptions` since there is no way to recover, usually. 
* The client (i.e. you) should always call the convenience method included on all responses: `response.isTdaError`

The error check is important because no calls throw checked exceptions, but instead `RuntimeExceptions` which
you may or may not check for. These are only called for errors we cannot recover from (bad internet connect, server is down, bad login, etc).

You should especially do this on the first call because if you cannot login (e.g. bad password), no further calls
will actually return valid data, just that all the responses will have `isTdaError=true` in the result.


## Integration Tests
Integration tests do require a TDA user and password, though are not needed to just build.

To run integration tests, you will need to rename this file *src/test/resources/com/studerw/tda/client/my-test.properties.changeme* to *my-test.properties* and add your own TDA user and pw.
Then run the following command.

```
mvn failsafe:integration-test
```

Don't worry - no purchases or transfers (to @studerw's account) will be made :/. Basicallyl we just check login and quote methods only.

## API Completed and TODO

### Authentication
* Login - DONE
* Logout - DONE
* KeepAlive - DONE

### Lookup 
* Quotes - DONE
* SymbolLookup - DONE
* PriceHistory - DONE
* VolatilityHistory - TODO
* OptionChain - PARTIAL
* BinaryOptionChain - TODO
* BalanceAndPosition - DONE
* OrderStatus - DONE
* PriceHistory - DONE

### Newes
* MarketOverview - TODO
* News - TODO
* FullStoryNews - TODO
* QuoteNews - TODO

### Trading
* EquityTrade - TODO
* OptionTrade - TODO
* EditOrder - TODO
* CancelOrder - TODO

### Conditional Trading
* ConditionalEquityTrade - TODO
* ConditionalOptionTrade - TODO

### Option Trading
* BuyWriteOptionTrade - TODO
* SpreadOptionTrade - TODO
* StraddleOptionTrade - TODO
* StrangleOptionTrade - TODO
* ComboOptionTrade - TODO
* MultiLegOptionTrade - TODO

### Saved Orders
* SavedOrders - TODO
* SaveEquityTrade - TODO
* SaveOptionTrade - TODO
* DeleteSavedOrders - TODO

### WatchLists
* GetWatchList - TODO
* CreateWatchList - TODO
* EditWatchlist - TODO
* DeleteWatchList - TODO

### Status
* BankingStatus - TODO
* CreateBankTransaction - TODO
* EditBankTransaction - TODO
* EditInternalTransaction - TODO
* DeleteBankTransaction - TODO

### Settings
* GetWebSettings - TODO
* EditWebSettings - TODO

### Streaming Data 
* All - TODO
* StreamerInfo - TODO
* MessageKey - TODO

## Issues / Todo
* pass around inputstreams instead of XML to parses, and get rid of XML strings stored in POJOs. 
* Need a validator for PriceHistory
* converting floats to BigDecimal causes loss of precision
* javadoc check and see if working correctly with package html

## How to Implement an API Call

* I have a copy of the API. I cannot post it here for copyright reasons, but it can be requested via email
or found via TDA dev support possibly. Most of the return types are almost exact copies 
of the new API here: [Restful API](https://developer.tdameritrade.com/).

* Each call in the API guide gives an example XML response and a table of the XML
schema type (e.g. what each field refers to and its general type).

* Take the example return XML and generate JAXB Pojo with the correct annotations.

* Modify the Pojo which means extend the `BaseTda` class, determine the error code,
modify String types to Java *Integer*, *BigDecimal*, possibly DateTime if it makes sense.

* Create the call within `TdaHttpClient` using the required parameters at a minimum
and write an accompanying integration test to ensure the call is successful with at least my
individual account. 

## Streaming API

* This API is significantly different than the other calls that it requires time
I currently do not have to implement. Hopefully, I will have time to finish it in the future.
