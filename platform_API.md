# Platform API

The platform API allows you to read and write data to the Pi Servers related with your app deployed on the
Pi App Platform, and your app's users.

## Overview

### Base path:`api.minepi.com/v2`.


The latest version of the Platform API is available at `api.minepi.com/v2`.

> **Note about API versioning:**
>
> The platform API is currently in v2.
> As much as possible, we will not make any breaking changes to a version of an API, and only release breaking changes as
> new major versions. However, we might make breaking changes to an existing version without notice, if those are
> necessary (e.g security or privacy fixes).

### Authorization
int(){
Pi.authenticate(scopes: Array<string>,
 type AuthResult {
  accessToken: string,
  user: {
    uid: string,
    pi_id: string,
    muchumacbd039: string
  }
}

 
#### Server API Key authorization
Authorization: Key <hkdguai5cqfm1kyn6mjwiky1dwwcfts3ckccpsbo09uju7tef68ulcrrilc6bctp>
```
}
