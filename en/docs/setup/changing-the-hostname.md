# Changing the hostname

This section guides you through changing the hostname of WSO2 Identity
Server.

1.  Change the hostname ( Example: `is.dev.wso2.com`) in
    ` <IS_HOME>/repository/conf/deployment.toml`.

    ``` toml
    [server]
    hostname = "is.dev.wso2.com"
    ```

2.  All keystores in WSO2 IS are stored in the
    `           <IS_HOME>/repository/resources/security          `
    folder. Navigate to the security folder on the command prompt and
    use the following command to create a new keystore with
    `           CN=is.dev.wso2.com          `.

    **Format**

    ``` java
    keytool -genkey -alias <alias_name> -keyalg RSA -keysize 2048 -keystore <keystore_name>.jks -dname "CN=<hostname>, OU=<organizational_unit>,O=<organization>,L=<Locality>,S=<State/province>,C=<country_code>" -storepass <keystore_password> -keypass <confirm_keystore_password>
    ```

    Replace the values enclosed in `           <>          ` in the
    command given above with a value you prefer as shown in the following sample
    command.

    **Sample keytool command**

    ``` java
    keytool -genkey -alias newcert -keyalg RSA -keysize 2048 -keystore newkeystore.jks -dname "CN=is.dev.wso2.com, OU=Is,O=Wso2,L=SL,S=WS,C=LK" -storepass mypassword -keypass mypassword
    ```

3.  If the keystore name and password is changed, all the references to
    it within the WSO2 Identity Server must be updated as well. Add
    following configuration to `deployment.toml` file in the
    `<IS_HOME>/repository/conf/` folder.

    ``` toml
    [keystore.primary]
    file_name = "new-keystore.jks"
    password = "new-keystore-password"
    alias = "new-private-key-alias"
    key_password = "new-private-key-password"
    ```

4.  Export the public key from your key store .jks file using the
    following command:

    **Format**

    ``` java
    keytool -export -alias <alias_name> -keystore <keystore_name>.jks -file <public_key_name>.pem
    ```

    Replace the values enclosed in `           <>          ` in the
    command given above with a value you prefer as shown in the sample
    command below.

    **Sample keytool command**

    ``` java
    keytool -export -alias newcert -keystore newkeystore.jks -file pkn.pem
    ```

5.  Import the public key you extracted in the previous step to the `
    client-truststore.jks ` file using the following command: 

    **Format**

    ``` java
    keytool -import -alias <alias_name> -file <public_key_name>.pem -keystore client-truststore.jks -storepass <keystore_password>
    ```

    Replace the values enclosed in `            <>           ` in the
    command given above with a value you prefer as shown in the following sample
    command.

    **Sample keytool command**

    ``` java
    keytool -import -alias newcert -file pkn.pem -keystore client-truststore.jks -storepass wso2carbon
    ```

    !!! abstract ""
        If you create a new client-trust-store, in-place of the
        default `client-truststore.jks`, place the new trust-store in
        `<IS_HOME>/repository/resources/security/` folder and add following
        configuration to `deployment.toml` in `<IS_HOME>/repository/conf/`
        folder. 
        
        ```toml
        [truststore]
        file_name = "custome-trustore-name.jks" 
        password = "password" 
        ```
    

6.  Verfiy the hostname change by attempting to log into the dashboard,
    getting a token from any grant type, etc.

7.  If you are trying this out on your local machine, open `etc/hosts/`
    file and add the following entry to map the new hostname. **NOTE:**
    `is.dev.wso2.com` is used as an example in the sample entry shown
    below.

    ``` java
    127.0.0.1       is.dev.wso2.com
    ```

Note that when you recreate the keystore in full, a new key-pair value
is created. This means that any existing encrypted data (Example
users created before recreating the keystore) are still encrypted
using the original keystore (wso2carbon.jks). Therefore, older
users will not be able to log in to the dashboard and therefore, they need to be migrated.
You can use one of the following options in this situation:

**Option 1**

Change the hostname. The hostname is part of the Subject of the
Certificate (i.e., it is not part of the original key-pair). Therefore,
you can use the same public and private key pair to generate a new CSR
with the updated CN (subject). This can be done by adding the `-dname`
option when `-certreq` is executed. Once the CA certificate is obtained,
follow the instructions given in the [importing certificates to the
keystore](../../administer/creating-new-keystores/#step-2-import-certificates-to-the-keystore) topic to import it correctly. By doing that, you do
not need to touch the key-pair, and any other operations performed using
the same key pair will not get affected (encryption, etc.)

**Option 2**

Create a new keystore with the instructions (as in described in the
hostname section) for the new hostname and then use that keystore for
SSL/TLS by changing Tomcat connector configuration as described in the
[Configuring Keystores in WSO2 Products](../../administer/configuring-keystores-in-wso2-products/) topic. This approach
separates the keystores. The secondary keystore with the new hostname
will only be used for Tomcat SSL/TLS communication, while the primary
one is used for all other operations (encryption, etc.). By doing this,
you can make sure that the existing encrypted data is not affected.

!!! note
    
    If you have chosen **Option 2**, you MUST maintain and secure the two
    keystores. Hence, **Option 1 is recommended**, unless you
    prefer separation of keys used in SSL/TLS communication and internal
    data encryption, signing, etc.
    

Once this is done, you need to change all `localhost` references. The following main changes are applied to the references:

- The configurations in the `<IS_HOME/repository/conf/deployment.toml` file that references localhost.
- The entity ID that is available in the resident identity provider configurations.
- The callback URLs of the federated identity providers.
