# EOSIO SDK for Java: RPC Provider Examples

EOSIO SDK for Java: RPC Provider contains an extensive set of functionality beyond the basics required for transactions.  The code snippets below show how to use some of this extended functionality.  It is important to note that these are simply example snippets and may not work the way you expect if you just copy and paste them into a method.  

Note: For clarity, some of these examples may use the soft key signature provider which is NOT recommended for production use!

## Extended RPC Provider Examples

### Get Account Information
This snippet retrieves information for an account on the blockchain.  There are several layers of response to unpack if all information is desired.  Some portions of the response are not fully unmarshalled, either due to size or because the responses can vary in structure.  These are returned as general `Map` objects.  The [NODEOS Reference](https://developers.eos.io/eosio-nodeos/reference) is helpful for decoding the parts of responses that are not fully unmarshalled.  

```java
try {
    EosioJavaRpcProviderImpl rpcProvider = new EosioJavaRpcProviderImpl(
            "https://my.test.blockchain");
    String testAccount = "test_account";

    RequestBody requestBody = RequestBody.create(okhttp3.MediaType.parse("application/json; charset=utf-8"), "{\"name\":\""+testAccount+"\"}");
    String response = rpcProvider.getAccount(requestBody);

    JSONObject jsonObject = (JSONObject)parser.parse(response);

    String account_name = (String) jsonObject.get("account_name");
    Double ramQuota = (Double) jsonObject.get("ram_quota");
    JSONArray permissions = (JSONArray) jsonObject.get("permissions");
    JSONObject permission = (JSONObject) permissions.get(0);
    String permissionName = (String) permission.get("perm_name");
    // Keep going for more information...
} catch (Exception ex) {
    println("Exception when calling getAccount(): " + ex.getLocalizedMessage()
            + "\n" + getStackTraceString(ex));
}
```

### Getting Transaction Information From the History Plugin

This snippet returns information on a transaction from the History API plugin.  Only a few fields are shown in the decoding example below.  The [NODEOS Reference](https://developers.eos.io/eosio-nodeos/reference) is helpful for decoding the parts of responses that are not fully unmarshalled. 

```java
try {
    EosioJavaRpcProviderImpl rpcProvider = new EosioJavaRpcProviderImpl(
            "https://my.test.blockchain");
    
    String getTransactionRequest = "{\n" +
            "\t\"id\": \"transaction id\",\n" +
            "\t\t\"block_num_hint\": 49420058\n" +
            "}";

    RequestBody requestBody = RequestBody.create(okhttp3.MediaType.parse("application/json; charset=utf-8"), getTransactionRequest);
    String response = rpcProvider.getTransaction(requestBody);

    JSONObject jsonObject = (JSONObject)parser.parse(response);
    String transactionId = (String) jsonObject.get("id");
    Long blockNum = (Long) jsonObject.get("block_num");
} catch (Exception ex) {
    println("Exception when calling getTransaction(): " + ex.getLocalizedMessage()
            + "\n" + getStackTraceString(ex));
}
```

### Retrieving Values From KV Tables

This snippet retrieves values from a KV table defined by a contract on the server.  The example below is requesting the values from the contract "todo" in the table named "todo".  It is querying the index named "uuid" for the value "bf581bee-9f2c-447b-94ad-78e4984b6f51".  The encoding type of the indexValue being supplied is "string".  Other valid values include: "bytes", "dec", "hex" and "name", depending on the index type.

```java
// Creating RPC Provider
IRPCProvider rpcProvider;
try {
    rpcProvider = new EosioJavaRpcProviderImpl("https://my.test.blockchain", ENABLE_NETWORK_LOG);
} catch (EosioJavaRpcProviderInitializerError eosioJavaRpcProviderInitializerError) {
    eosioJavaRpcProviderInitializerError.printStackTrace();
    println(Boolean.toString(false), eosioJavaRpcProviderInitializerError.getMessage());
    return null;
}

String getKvTablesRequestJson = "{\n" +
            "    \"json\" : false\n" +
            "    \"code\" : \"todo\"\n" +
            "    \"table\" : \"todo\"\n" +
            "    \"encode_type\" : \"string\"\n" +
            "    \"index_name\" : \"uuid\"\n" +
            "    \"index_value\" : \"bf581bee-9f2c-447b-94ad-78e4984b6f51\"\n" +
            "    \"reverse\" : false\n" +
            "}";

    // If the RPC Provider was declared as the IRPCProvider interface type you have to cast it.
    EosioJavaRpcProviderImpl fullRpcProvider = (EosioJavaRpcProviderImpl)rpcProvider;

    RequestBody requestBody = RequestBody.create(okhttp3.MediaType.parse("application/json; charset=utf-8"), getKvTablesRequestJson);
    try {
        println("Requesting KV table rows...");
        String response = fullRpcProvider.getKvTableRows(requestBody);
        JSONParser parser = new JSONParser();
        JSONObject jsonObject = (JSONObject)parser.parse(response);
        JSONArray jsonArray = (JSONArray)jsonObject.get("rows");
        String serializedResult = (String)jsonArray.get(0);
        println("Your row result (serialized) is:  " + serializedResult);
    } catch (Exception getKvTableRowError) {
        this.publishProgress(Boolean.toString(false), "Error getting table rows: " + getKvTableRowError.getLocalizedMessage());
    }
```