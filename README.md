# shoal - an ergonomic configuration file format

<!-- panels:start -->
<!-- div:left-panel -->
`shoal` is a simple and readable `INI`-like format, supporting hierarchical structures with an arbitrary number of nesting levels.   

```shoal
;shoal supports comments,

root_dir = ~/Photos                ; parameters, 
supported_files = [*.jpg, *.png]   ; arrays,            
#thumbnails:                       ; structures,
  enabled = true
  max_width = 128
  max_height = 128
---                    
#shared_albums:                     ; and arrays of structures
###
  dir   = summer_2019
  title = "Summer (2019)"  
###    
  dir   = misc
  title = Misc      
```
<!-- panels:end -->

<!-- panels:start -->
<!-- div:left-panel -->
`shoal` structure can be described in terms of a document tree: nodes can contain named values (parameters), named lists of values (arrays), named nodes (structures) and named lists of nodes (arrays of structures). The top level of the config file acts as the unnamed root node.  
To represent levels of this tree `shoal` uses opening and closing tokens similar to opening and closing tags in `XML` or braces in `JSON`, indenting levels is recommended, but not required.  When node is created with an opening token `#nodeName:` the context descents on its level and all the following elements end up being the children of this node until it's closed: 
<!-- panels:end -->

<!-- panels:start -->
<!-- div:left-panel -->
```shoal
#photos:
  root_dir = ~/Photos  
  
#videos:
  root_dir = ~/Videos  
```
<!-- div:right-panel -->
<!-- tabs:start -->
#### **JSON**
```json
{
  "photos" : {
      "root_dir" : "~/Photos",
      "videos" : {
          "root_dir" : "~/Videos"
      }
  }
}
```
#### **YAML**
```yaml
photos:
  root_dir: ~/Photos
  videos:
    root_dir: ~/Videos      
```
#### **TOML**
```toml
[photos]
  root_dir = "~/Photos"
  [photos.videos]
    root_dir = "~/Videos"
```
<!-- tabs:end -->
<!-- panels:end -->

<!-- panels:start -->
<!-- div:left-panel -->
Here `videos` is a child node of `photos`, because the `photos` node is closed implicitly at the end of the document.

To fix this oversight and add `videos` node on the same level as `photos` we need to close the `photos` node first. This can be done with either of 3 possible closing tokens: 
   * `---` which closes the node and returns the context back to the root node;
   * `-` which closes the node and returns the context to its parent node;
   * `--photos` which closes the node specified by the provided name and returns the context to its parent node;
<!-- panels:end -->

<!-- panels:start -->
<!-- div:left-panel -->
```shoal
#photos:
  root_dir = ~/Photos  
---  
#videos:
  root_dir = ~/Videos  
```
<!-- div:right-panel -->
<!-- tabs:start -->
#### **JSON**
```json
{
  "photos": {
    "root_dir" : "~/Photos"
  },
  "videos": {
    "root_dir" : "~/Videos"
  }
}
```
#### **YAML**
```yaml
photos:
  root_dir: ~/Photos
videos:
  root_dir: ~/Videos
```
#### **TOML**
```toml
[photos]
  root_dir = "~/Photos"
[videos]
  root_dir = "~/Videos"
```
<!-- tabs:end -->
<!-- panels:end -->

<!-- panels:start -->
<!-- div:left-panel -->
Simple configs can be written with only `-` or `---` closing tokens. In multilevel configs `--nodeName` format can be used when you need to close several nodes at the same time without returning to the root node: 
<!-- panels:end -->

<!-- panels:start -->
<!-- div:left-panel -->
```shoal
#photo_server:
  #security:
    password_strength = 8
    #password_policy:     
      length = 4       
    -
    #user-lock-policy:
      attempts = 0
      password_expiry_days = 0
  --security
  #host:
    address=127.0.0.1
    port = 8080
```
<!-- div:right-panel -->
<!-- tabs:start -->
#### **JSON**
```json
{
  "photo_server":{
    "security":{
      "password_strength": 8,
      "password_policy":{
        "length" : 4
      },
      "user-lock-policy":{
        "attempts": 0,
        "password_expiry_days": 0
      }
    },
    "host":{
      "address":"127.0.0.1",
      "port": 8080 
    }
  }
}
```
#### **YAML**
```yaml
photo_server:
  security:
    password_strength : 8
    password_policy:
      length : 4
      
    user-lock-policy:
      attempts: 0
      password_expiry_days: 0
  host:
    address: 127.0.0.1
    port: 8080
```
#### **TOML**
```toml
[photo_server]
  [photo_server.security]
    password_strength = 8
    [photo_server.security.password_policy]
      length = 4
    
    [photo_server.security.user-lock-policy]
      attempts = 0
      password_expiry_days = 0
  [photo_server.host]
    address = "127.0.0.1"
    port = 8080
```
<!-- tabs:end -->
<!-- panels:end -->

<!-- panels:start -->
<!-- div:left-panel -->
If possible it's better to minimize the usage of different closing tokens to keep the structure uniform. For example, the same config can be reordered to avoid the explicit closing of `security` node:
<!-- panels:end -->

<!-- panels:start -->
<!-- div:left-panel -->
```shoal
#photo_server:
  #host:
    address = 127.0.0.1
    port = 8080
  -  
  #security:
    password_strength = 8
    #password_policy:
      length = 4
    -
    #user-lock-policy:
      attempts = 0
      password-expiry-days = 0
```
<!-- div:right-panel -->
<!-- tabs:start -->
#### **JSON**
```json
{
  "photo_server":{
    "host":{
      "address":"127.0.0.1",
      "port": 8080 
    },
    "security":{
      "password_strength": 8,
      "password_policy":{
        "length" : 4
      },
      "user-lock-policy":{
        "attempts": 0,
        "password_expiry_days": 0
      }
    }
  }                              
}
```
#### **YAML**
```yaml
photo_server:
  host:
    address: 127.0.0.1
    port: 8080
  security:
    password_strength : 8
    password_policy:
      length : 4
      
    user-lock-policy:
      attempts: 0
      password_expiry_days: 0
```
#### **TOML**
```toml
[photo_server]
  [photo_server.host]
    address = "127.0.0.1"
    port = 8080
  [photo_server.security]
    password_strength = 8
    [photo_server.security.password_policy]
      length = 4
    
    [photo_server.security.user-lock-policy]
      attempts = 0
      password_expiry_days = 0
```
<!-- tabs:end -->
<!-- panels:end -->

<!-- panels:start -->
<!-- div:left-panel -->
The last fairly unique looking feature of `shoal` is the arrays of structs. To create them add `###` on the next line after the opening token before the first element, and before all following elements as a separator. To close them it's possible to use all the closing tokens described earlier. 
<!-- panels:end -->

<!-- panels:start -->
<!-- div:left-panel -->
```shoal
#video_server:
  #hosts:
  ###
    address = 127.0.0.1
    port = 8081
  ###
    address = 127.0.0.1
    port = 8082
  -
  root_dir=~/Videos        
```
<!-- div:right-panel -->
<!-- tabs:start -->
#### **JSON**
```json
{
  "video_server":{
    "hosts":[
      {
        "address" : "127.0.0.1",
        "port": 8081
      },
      {    
        "address": "127.0.0.1",
        "port": 8082
      }
    ],
    "root_dir" : "~/Videos"
  }
}       
```
#### **YAML**
```yaml
video_server:
  hosts:
    - address : 127.0.0.1
      port: 8081
         
    - address: 127.0.0.1
      port: 8082      
  root_dir: ~/Videos
```
#### **TOML**
```toml
[video_server]
  root_dir = "~/Videos"
  [[video_server.hosts]]
    address = "127.0.0.1"
    port = 8081
  [[video_server.hosts]]       
    address = "127.0.0.1"
    port = "8082"        
```
<!-- tabs:end -->
<!-- panels:end -->

## Motivation
<!-- panels:start -->
<!-- div:left-panel -->
`shoal` was designed with the following goals in mind:
* `(A)` the declaration of parameters and structures shouldn't use the same syntax;
* `(B)` multilevel structures should be supported with direct representation of hierarchy;
* `(C)` the indentation of config levels should be optional;
* `(D)` the syntax noise level should be as low as possible (this of course is subjective).

Let's see how `shoal` covers these requirements in comparison to popular solutions for software configuration:

  |     | shoal | JSON  | YAML  | TOML  | INI   | XML   |
  |-----|-------|-------|-------|-------|-------|-------|
  | A   | +     | -     | -     | +     | +     | +     |
  | B   | +     | +     | +     | -     | -     | +     |
  | C   | +     | +     | -     | +     | +     | +     |
  | D   | +/-   | -     | +     | +/-   | +     | -     |

These are the personal requirements of the author of `shoal` who not surprisingly belongs to the tribe of people preferring `XML` to `JSON`. As you can see the closest contenders are the `INI` format that doesn't match the important functional requirement, and `XML` which is too verbose and feels like an overkill for simple config files.  

In other words, `shoal` aims to be an `INI` replacement with nesting structures support. Like `INI` it's intended solely for software configuration, `shoal` is implementation-dependent and its specification is too vague to be used as a general data exchange format reliably.

<!-- panels:end -->

## Implementations
* C++
  * [`figcone`](https://github.com/kamchatka-volcano/figcone)
