# A guideline to ecapture use
In this section we cover all the required information for getting started with ecapture.

---
## What is ecapture ?
- eCapture is a tool for capturing HTTPS plaintext packets without any CA certificate based on eBPF and requires root privileges
---
## Requirements before installation
- Linux kernel version >= 4.18 is required.
- Enable BTF [BPF Type Format (BTF)](https://www.kernel.org/doc/html/latest/bpf/btf.html) (Optional, 2022-04-17)
- Using the following command you can assure whether the current kernel version is BTF support or not 
``` bash
cat /boot/config-`uname -r` | grep CONFIG_DEBUG_INFO_BTF
```
- `CONFIG_DEBUG_INFO_BTF=y` would be the expected output
---
## Installation process
- The installation process is relatively easy. All you need is just to download the latest release based on your arcihtecture from [here](https://github.com/gojue/ecapture/releases)

> [!NOTE] Note
> Note that not all releases are compatible with your system architecture. So you may want to search among releases to find the desired one
> 

---
## How to use
### TLS module
- First we need to find out which encryption library is used by the target process. For example by using the following command we can find the encryption libraries used by curl command
``` bash
ldd `which curl` | grep -E "tls|ssl|nss|nspr"
```
- if the mentioed command did not provide you with corresponding encryption library. you should use this command:
```bash
pldd 7590 | grep -E "tls|ssl|nss|nspr"
```

> [!NOTE] Note
> you have to substitute the number with actual pid of the process
- Then, using this command we can capture encrypted traffic
```bash
sudo ./ecapture tls --libssl="/lib/x86_64-linux-gnu/libssl.so.3"
```
```bash
sudo ./ecapture nss --nspr="/usr/lib/firefox/libnspr4.so"
```
### MYSQL module
```bash
sudo ./ecapture mysql
```
### GOLANG TLS module
Here is code for a golang https client
```go
package main

import (
	"crypto/tls"
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
)

func main() {
	// Specify the URL to request
	url := "https://www.example.com"

	// Create a custom transport with default TLS configuration
	tr := &http.Transport{
		TLSClientConfig: &tls.Config{MinVersion: tls.VersionTLS12},
	}

	// Create a client with the custom transport
	client := &http.Client{Transport: tr}

	// Create the request
	req, err := http.NewRequest("GET", url, nil)
	if err != nil {
		log.Fatalf("Failed to create request: %v", err)
	}

	// Perform the request
	resp, err := client.Do(req)
	if err != nil {
		log.Fatalf("Failed to perform request: %v", err)
	}
	defer resp.Body.Close()

	// Read the response body
	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		log.Fatalf("Failed to read response body: %v", err)
	}

	// Print the response status and body
	fmt.Printf("Response Status: %s\n", resp.Status)
	fmt.Printf("Response Body: %s\n", body)
}

```
next step is to build this golang application
```bash
go build -o https_client
```

> [!NOTE] Note
> note that the binary file should contain the debug info. because this is how ecapture detemines the address of used tls library

- for further check you can use this command to build your golang file to realize whether ecapture works properly or not
```bash
go build -ldflags="-s -w" -o https_client
```
- These commands actually help you to realize wheter the binary file is stripped or not 
```bash
objdump -t https_client
file https_client
```
