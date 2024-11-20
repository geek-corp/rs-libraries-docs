# TVS Axios Lib

## Description
TVS Axios Lib is a TypeScript library that provides a method to create a custom Axios instance with flexible configuration options, allowing for encryption, dynamic headers, and more. It extends Axios with a convenient way to merge configurations and handle common HTTP requests with customized settings.

## Features
* Custom Axios instance creation
* Support for dynamic headers
* Optional encryption configuration for requests
* Easy merging of configurations for different request methods (`GET`, `POST`, `PUT`, `DELETE`, etc.)
* Safe checks to ensure encryption keys are set when encryption is enabled

## Installation
To install the library, use Yarn:

```bash
    yarn add @geek-corp/tvs-axios-lib
```

## Configure Access Private Packages
Configuring the `.npmrc` File: To access packages from GitHub Packages, you need to configure the `.npmrc` file in the root of your project. Follow these steps:

* Create a file named `.npmrc` in the root of your project (next to `package.json`).
* Add the following lines to the `.npmrc` file:
    ```bash
    @geek-corp:registry=https://npm.pkg.github.com
    //npm.pkg.github.com/:_authToken=${GITHUB_TOKEN}
    ```
* Make sure to replace ${GITHUB_TOKEN} with your personal GitHub authentication token. You can generate a token by following these instructions:

    1. Go to [GitHub Settings](https://github.com/settings/tokens).
    2. Create a new token with `read:packages` access.
    3. Copy the token and paste it into the `.npmrc` file.

### Important Note: 
Do not commit the `.npmrc` file to your repository for any reason, as it may contain sensitive information like your authentication token. Instead, add .npmrc to your `.gitignore` file to prevent it from being tracked.

## Usage
### Import the `createHttpClient` method
```typescript
import { createHttpClient } from '@geek-corp/tvs-axios-lib';
```

### Import the `CustomAxiosHeaders` method
```typescript
import { CustomAxiosHeaders } from '@geek-corp/tvs-axios-lib';
```

### Create a custom HTTP client

You can create a new Axios instance by passing a custom configuration. If encryption is enabled, you must also provide a `privateKey` and `publicKey`.

```typescript
import { createHttpClient, CustomAxiosHeaders } from "@geek-corp/tvs-axios-lib";

const httpClient = createHttpClient({
  baseURL: 'http://your-domain:3001/', // Optional, set to your base url api
  enableEncryption: true,  // Optional, set to false by default
  privateKey: 'your-private-key',
  publicKey: 'your-public-key',
  headers: CustomAxiosHeaders.from({
    'Authorization': 'Bearer your-access-token',
  })
});
```
### Making requests

You can use the custom HTTP client to make various HTTP requests (GET, POST, PUT, DELETE, etc.). The method automatically merges the provided configuration with any new configuration passed during the request.

#### GET Request
```typescript
httpClient.get('/api/resource').then(response => {
  console.log(response.data);
}).catch(error => {
  console.error(error);
});
```

#### POST Request
```typescript
httpClient.post('/api/resource', { key: 'value' }).then(response => {
  console.log(response.data);
}).catch(error => {
  console.error(error);
});
```

#### PUT Request
```typescript
httpClient.put('/api/resource/123', { key: 'updatedValue' }).then(response => {
  console.log(response.data);
}).catch(error => {
  console.error(error);
});
```

#### DELETE Request
```typescript
httpClient.delete('/api/resource/123').then(response => {
  console.log(response.data);
}).catch(error => {
  console.error(error);
});
```

## Method Overview
`createHttpClient(config?: CustomAxiosConfig): AxiosInstance`
This method creates a new Axios instance based on the custom configuration you provide.

#### Parameters
* `config` (optional): A CustomAxiosConfig object that can include options like:
    * `enableEncryption`: Boolean value to enable encryption for requests.
    * `privateKey`: The private key used for encryption.
    * `publicKey`: The public key used for encryption.
    * `headers`: Custom headers to include with every request.

#### Returns
An Axios instance with the customized configuration.

#### Throws
An error will be thrown if `enableEncryption` is set to true but `privateKey` or `publicKey` is missing:
```typescript
throw new Error("enableEncryption is true but privateKey or publicKey not set. Please set these properties.");
```
### Request Methods
Each of the following HTTP methods are extended from Axios and support the custom configuration merging:

* `instance.get<T, R>(url: string, config?: Partial<CustomAxiosConfig>)
`
* `instance.post<T, R>(url: string, data?: T, config?: Partial<CustomAxiosConfig>)
`
* `instance.put<T, R>(url: string, data?: T, config?: Partial<CustomAxiosConfig>)
`
* `instance.delete<T, R>(url: string, config?: Partial<CustomAxiosConfig>)
`
* `instance.patch<T, R>(url: string, data?: T, config?: Partial<CustomAxiosConfig>)
`
* `instance.options<T, R>(url: string, config?: Partial<CustomAxiosConfig>)
`
## Configuration Merging
The `mergeConfig` function merges the base configuration (passed when creating the client) with any additional configurations passed during individual requests.

```typescript
const mergeConfig = (newConfig?: Partial<CustomAxiosConfig>) => {
  const mergedHeaders = {
    ...config?.headers,
    ...newConfig?.headers
  };

  return {
    ...config,
    ...newConfig,
    headers: mergedHeaders,
  };
};
```
This ensures that each request can have its own specific configuration while still retaining the overall global configuration.

## Error Handling
* If you attempt to enable encryption without setting both `privateKey` and `publicKey`, an error will be thrown.
* Standard Axios error handling is retained, meaning `catch` can be used to handle any request failures.

## Verify Checksum with Hash In The Server
Verify the integrity of the data sent by comparing a hash (checksum) of the decrypted data with a hash that was previously signed and sent in a JWT token. This ensures that the data hasn't been tampered with during transit.

### Prerequisites
Before using this verification, ensure you have installed the `@geek-corp/tvs-crypto-lib` package in your project:
```bash
yarn add @geek-corp/tvs-crypto-lib
```

Additionally, make sure to configure the HTTP client correctly (set `enableEncryption` in `true`) as shown below:
```typescript
const httpClient = createHttpClient({
  enableEncryption: true,
  ....
});
```

### Request
* Headers:
  * `x-request-id`: A unique key that can be used for tracking purposes.
  * `x-strcode`: A hash representing the checksum of the original data, used to verify the integrity of the decrypted data.
* Body:
  * The encrypted data sent from the client. This data will be decrypted using the server's private key.

### Flow of the Verification Process
1. **Hash Verification**: The server retrieves the hash from the `x-strcode` header. This hash represents the checksum of the original data.

    ```typescript
    const hash = req.headers['x-strcode'] as string;
    ```

2. **Decrypt Data**: The server decrypts the encrypted data sent in the request body using the private RSA key.
    ```typescript
    const data = decryptData(req.body, privateKey);
    ```
3. **Generate Hash (Checksum) from Decrypted Data**: After decrypting the data, the server generates a new hash (checksum) from the decrypted data.
    ```typescript
    const generatedHash = generateHash(data);
    ```
4. **Compare Hashes**: The server compares the generated hash from the decrypted data with the hash retrieved from the `x-strcode` header. If the hashes match, it confirms that the data has not been tampered with.
    ```typescript
    console.log(generatedHash === hash);
    ```
### Flow Tracking Request Id
The `x-request-id` header can be used for tracking purposes, allowing the server to log or monitor requests based on unique identifiers.
  ```typescript
    const trackUuid = req.headers['x-request-id'] as string;
  ```

## Contributing
Contributions are welcome. Please open an issue or pull request in the GitHub repository.

## License
This project is licensed under the ISC license. See the LICENSE file for more information.

## Issues
If you encounter any issues, please report them on the [GitHub repository](https://github.com/geek-corp/tvs-axios-lib/issues).
