# Radar24 API Reverse Engineering

This Repository Contains the Entire Report on how the Radar24 App Communicates & accesses the 
Radar Information for each country. This Reports provides a full insight into all of the findings and how to call them w/ *Cookies & Session Data*.  

To Reverse Engineer the Communications following App, I have performed SSL Pinning Bypass using Frida and Burpsuite Certificate to intercept all SSL Communications b/w the RadarBot24 App and Backend Servers. Below are all of the findings & all communications made.

## Validating Traffic Stops

To validate red light stops or police radars, you need to go ahead and add the correct
`id_radar` and confirm if that `id_radar` exists in that country. In the following case
we are validating a German Radar using the RadarBot24 API. 


```python
import requests

session = requests.session()

burp0_url = "https://www.radarbotservices.com:443/ws_confirm_radar.php"

burp0_headers = {"Content-Type": "application/x-www-form-urlencoded; charset=UTF-8", "User-Agent": "Dalvik/2.1.0 (Linux; U; Android 12; Pixel 3 XL Build/SQ1D.220205.004)", "Accept-Encoding": "gzip, deflate, br"}

burp0_data = {"ws_version": "2", "rb_version": "189", "password": "radar_asm256K!@", "pais": "ru", "id_dispositivo": "9B23-331B-3624-7873", "p": "DLnadhl3GpW7YhRhCGoLE38oWwJtkPaI/JcEFvEDEwh8XF32+twT", "os": "0", "up": "0", "id_firebase": "fnwo9KelRL6FbNt6wIKYQ2:APA91bE4wn_huabwvo32eXvB4e0uG0DePUueZYVDQEOUGy_M1RSS2UoajIPHC6B947j61NwEpxBqKSaE_5HyTnpXWfysyYDkQF45L_xBj_nrl8-wFZPzqcZIavjAjmC2BbMGfEpG6Xsd", "latitud": "55.743866", "longitud": "37.654709", "id_radar": "294675", "tipo": "0", "correcto": "1", "ts": "0&"}

session.post(burp0_url, headers=burp0_headers, data=burp0_data)
```

### JSON Responses

**Successfully Validated**:

```json
{"code":0,"message":"CONFIRM_FIX_RADAR_SUCCESS"}

```

**Failed to Validate (Radar ID does not exist)**:

```json
{"code":0,"message":"INVALID RADAR ID"}
```


## Getting Alerts with Lat/Long Co-ordinates

The RadarBot Application tracks user repored alerts for Stop Signs and in Lat/Long Co-ordinates using the following API. Below we are testing at a know location for a 
red light. Please do note that this API requires you to precisely pass the lat/long on
the road to get the alert.

If you pass a co-ordinate where there are no alerts nearby then or exactly infront of it 
the apps behaviour in my testing does not produce it. This only works when you are really near
it or there have been some user reports about something.

```python
import requests

session = requests.session()

burp0_url = "https://www.radarbotservices.com:443/ws_get_alerts.php"

burp0_headers = {"Content-Type": "application/x-www-form-urlencoded; charset=UTF-8", "User-Agent": "Dalvik/2.1.0 (Linux; U; Android 12; Pixel 3 XL Build/SQ1D.220205.004)", "Accept-Encoding": "gzip, deflate, br"}

burp0_data = {"ws_version": "2", "rb_version": "189", "password": "radar_asm256K!@", "pais": "de", "id_dispositivo": "9B23-331B-3624-7873", "p": "D9u45omuw8fpRjDOpytK2rZFNqzDgefBtfTaIVPnhr+H3KUpunS8N8nk", "os": "0", "up": "0", "id_firebase": "fnwo9KelRL6FbNt6wIKYQ2:APA91bE4wn_huabwvo32eXvB4e0uG0DePUueZYVDQEOUGy_M1RSS2UoajIPHC6B947j61NwEpxBqKSaE_5HyTnpXWfysyYDkQF45L_xBj_nrl8-wFZPzqcZIavjAjmC2BbMGfEpG6Xsd", "latitud": "52.314590", "longitud": "13.601879", "ts": "0&"}

session.post(burp0_url, headers=burp0_headers, data=burp0_data)
```


### JSON Responses

**Alerts Found**:

```json
{"code": 0,"message":"GET ALERTS OK","alerts_near":[{"id":"73734562","t":"01:53:24","st":"01:53:24","lt":"01:53:24","la":"52.403144","lo":"13.543976","ta":"6","txt":"","ad":"Chorweilerstra\u00dfe","ok":null,"ko":null,"f":"1.0000"}],"myalerts":[]}
```

**No Alerts to Report**:

```json
{"code": 4,"message": "NO ALERTS IN AREA"}
```

## Getting the App License Keys

The App Communicates with the Server to Get License Keys, I believe that the decrpytion
key for the encrpyted database, could be in `access_key_secret`. But that is out of the
premise of this project. You would need to hire a Pentration Tester w/ Android App Reverse
Engineering Experience to see which crpytographic algorithm is being used to access the da
tabase, it may be hardcoded inside. 


```python
import requests

session = requests.session()

burp0_url = "https://www.radarbotservices.com:443/ws_get_licenses.php"

burp0_headers = {"Content-Type": "application/x-www-form-urlencoded; charset=UTF-8", "User-Agent": "Dalvik/2.1.0 (Linux; U; Android 12; Pixel 3 XL Build/SQ1D.220205.004)", "Accept-Encoding": "gzip, deflate, br"}

burp0_data = {"ws_version": "2", "rb_version": "189", "password": "radar_asm256K!@", "pais": "de", "id_dispositivo": "9B23-331B-3624-7873", "p": "Cxl6XzDDrsLsDnhWP43sHnI2RQZpdBJtcbGjajf/fFlyK38tpqc=", "os": "0", "up": "0", "id_firebase": "fnwo9KelRL6FbNt6wIKYQ2:APA91bE4wn_huabwvo32eXvB4e0uG0DePUueZYVDQEOUGy_M1RSS2UoajIPHC6B947j61NwEpxBqKSaE_5HyTnpXWfysyYDkQF45L_xBj_nrl8-wFZPzqcZIavjAjmC2BbMGfEpG6Xsd", "ts": "0&"}

session.post(burp0_url, headers=burp0_headers, data=burp0_data)
```

### JSON Responses


```json
{
  "free": {
    "app_id": "0000",
    "app_code": "0000",
    "license_key": "0000",
    "api_key": "LA-8LcNiUu60QQxoAFI3TsuAeSsuUDmaogNUZRqYeLM",
    "access_key_id": "EdilteF9NY1wki2FNtehNA",
    "access_key_secret": "iGhjbsE8U_uGYQ5NEAdyxqdgsC5qQ1srnHrNo_0n9tbke8iBNPvRsfNUq7gLTQo911iMTUFTIQaH4GNb71e18g"
  },
  "pro": {
    "app_id": "0000",
    "app_code": "0000",
    "license_key": "0000"
  },
  "trial": {
    "app_id": "5999yqGLg5SsCVCVSlvr",
    "app_code": "viRoQJdkrOHOXKwS3Q5fHw",
    "license_key": "OIKVzxweXdwpb1V7VWxjYAADmf0/3xOWfdwcbE8KVcYFuq4vt9z1VXVJNc4K/RTI++YxTT2RhlsiDm4rmEaQvdk8KSZj4SRnpb2OBCJv6KgqvZjfgTk7qGwF2B/KJst1Ez72aOY6n3ydHd4FXI1wW80yW2p10jcwDrLps/3qrwudz70JaiD3idvT0hxjbxIC9jK6ZGNx4nLyGEjBCcd4Wb28g8j5CLSD1eoY9vifDwoYt3bVo+UKDtKiWey4ArFp2g6kKcUC0CIni9/S1Dg+x89LmHSKWuBQfMseoc/GyTrOVcerFJVG3vDfbw+m4ksQTGlMvEKxisx5Tk16cEuabteoYg+ij5se/J2ge7kDE7F2TsSH1rCSpJx+SY7sKYYlurFTx6+rxXIt3uGURBzu8IjgBNsPzQbNUiqKrniCJ6I06K1yrj8JfAhH+/bzHNByJVLBpohdYLRQTBMMtJQEqWwym7HUsJgcKq9vW4/1btPnAZfSDF5StmXsavcget/S3CJCdfFU2hFm1i1GqA72lWDBqhCUwHQ3U0bzgqo90SR30NINqU8ijBwF6u+tDpViVYbE2sTimfWHpWQMnCSvmHkymNSrmoPuNmQVCkdyCC7SiWxtYRY/KSRGXmqHkzSAsT7S/X/YSOQrl/dIqJBahz3Nv5Yuv1JAJs8jl/Huv/w="
  },
  "suscriber": {
    "app_id": "0zJ2whA1UmIt9r4XRhvd",
    "app_code": "GPxrQmp4Vg5aO3MR3hpNCQ",
    "license_key": "h75wzaiCUWaB7jN2d1xDqQcPSQUxjuNBGw0he1Nhkf/u9/xeZcuxwGU7XyO0xflv+NIrJRY4+LAZ5cV4HANXVbChDu3ZXMSo02gUKQIh+vTHkRxZdl+19zXrg+l1k2EjSrMw9a0auAXKj6gkVbnYrAA6uodG7AzyVTnNfxUhZWnv5wJjBpbEY3VskpZg5bijkr4U1/o76DcHGNmYhkpQE5H0QSB1Wt9uyvuOy5m9LxTfibB7cFo25GEwh0DN8FUtQOXGEjxeYAVb3DkDV3q9t6Xk6xtJ6X3g1wKYDODXnzEigaZkBL+gos8WL9KDR+rdv7cV0jcDrQgwcPJHb70GqS4CHIKZmZt429NeiLNYbx6tED6wVHwkly7dQmOR2z2U6xvzIgc4KCpo2vmORSlvjAnzSuDYcbG5kmW6tCwgoprkD8oZ6LDUP7cwASLnsgCCumDapAMg/gAyElOpkNXgDnAlmW6W6YoxRhlEx7AB7Cj+fRwu32Up2vdOTsvpJLkHd641IilKd2kCvrl0s6gFbGoT0lTgfZEsh2wFjdlUIHb6s1yUvKT1uq6RCV3tSuaq+Ev7nfaCHuiPyBaD4TXzTEPosw2BVanFpPcYilbmfqboDCgDP3UjVnKoLB8MLeJuGnqbsfpd6ZXTzgsnxT6tgvgw0znO6EBMaeur3n9yDC0=",
    "api_key": "LA-8LcNiUu60QQxoAFI3TsuAeSsuUDmaogNUZRqYeLM",
    "access_key_id": "CtHXzBjGlT80KR1t-13QDQ",
    "access_key_secret": "3Dem3RiUT1UjT74VqgxxdF7Zs31x0rBvocLVE0OR5q73rNuonvXoGThXOEru7ai09mDiGApl_iEAaPjWuQnbQw"
  },
  "truck_ev": {
    "app_id": "0zJ2whA1UmIt9r4XRhvd",
    "app_code": "GPxrQmp4Vg5aO3MR3hpNCQ",
    "api_key": "LA-8LcNiUu60QQxoAFI3TsuAeSsuUDmaogNUZRqYeLM",
    "access_key_id": "1d6lmt8CllgR8fFnAQ96lw",
    "access_key_secret": "2U5Dm-NPTGKsV16bZM8v4g9JqxQcXthP7BAqMtlbOvlSLH40C_vmP4zzFnNMEM5MGelTImNgCP-FSgVRg68qlQ"
  }
}
```

### Getting Radar / Red Light Distances

The Radar24 Application Gathers the Radar Information from the database and then displays
it into the Google Maps. Here, you can go ahead and see how the application computes
when to display the alert when the distance gets to a certain point. As it loops over
the `current_point` which is passed as a parameter. `engine` currently selected is `grasshopper` for GPS devices that jump from one place to another.


```python
import requests

session = requests.session()

burp0_url = "https://www.radarbot-osm.com:443/ws_radardist.php?point=52.3241751521,13.73274754,52.3233004986,13.7084197757,52.3233004986,13.7084197757,52.3233004986,13.7084197757,52.3233004986,13.7084197757,52.3190860166,13.6907268561,52.3190860166,13.6907268561,52.3190860166,13.6907268561,52.3152687809,13.6745950766,52.3152687809,13.6745950766&accuracy=0,0,0,0,0,0,0,0,0,0&heading=92,92,92,73,73,73,73,73,73,73&dest=52.323381,13.75139&engine=graphhopper&current_point=6"

burp0_headers = {"User-Agent": "Dalvik/2.1.0 (Linux; U; Android 12; Pixel 3 XL Build/SQ1D.220205.004)", "Accept-Encoding": "gzip, deflate, br"}

session.get(burp0_url, headers=burp0_headers)
```

### JSON Response

**Current Point (6)**:
```json
{"code":"ok", "d":[9312,0,9312,0], "n":[7,7]}
```

**Current Point (0)**:

```json
{"code":"ok", "d":[6343,0,6343,0], "n":[7,7]}
```


# Accessing Radars/Traffic Lights/Alerts/Police Reports

The RadarBot24 App (`com.vialsoft.radarbot_free`) is going ahead and communicating w/ it's
server to do the following tasks.


### Application Workflow

1. Checks to See if it's in a New Country.
2. If it's in a New Country it Requests the Download URL for the Database.
3. Fetches the Encrypted Database from the URL, Dencrpyts it and Displays on the Screen.
4. Sends Occasional Pings to See if the Database Needs to be Updated.

## Requesting the RadarBot24 Database URL

Below I am requesting the Database URL for Germany. You need the following Parms to
get the URL, `Country Code` & `Radar Version`.

**Key Takeaways**:

1. Each Country Has it's Own Database.
2. It's Encrypted (Not using SSL) - I have bypassed it but using some other technique.
3. Good News is that I have not seen the app communicating a key, so it's either hard
coded or it's somewhere in the `License Key` section.
4. We need to Hire a Android Application Reverse Engineering Professional to Proceed Forward
to see how the app is decrypting the database.

```python

import requests

session = requests.session()

burp0_url = "https://www.radarbotservices.com:443/ws_get_database_url.php"
burp0_headers = {"Content-Type": "application/x-www-form-urlencoded; charset=UTF-8", "User-Agent": "Dalvik/2.1.0 (Linux; U; Android 12; Pixel 3 XL Build/SQ1D.220205.004)", "Accept-Encoding": "gzip, deflate, br"}

burp0_data = {"ws_version": "2", "rb_version": "189", "password": "radar_asm256K!@", "pais": "ru", "id_dispositivo": "9B23-331B-3624-7873", "p": "CrLR+pVLJvnXs8VZMMipOFQjUMSr1I5li9I9qUpBq8alLYvWPA==", "os": "0", "up": "0", "id_firebase": "fnwo9KelRL6FbNt6wIKYQ2:APA91bE4wn_huabwvo32eXvB4e0uG0DePUueZYVDQEOUGy_M1RSS2UoajIPHC6B947j61NwEpxBqKSaE_5HyTnpXWfysyYDkQF45L_xBj_nrl8-wFZPzqcZIavjAjmC2BbMGfEpG6Xsd", "db_country": "de", "ts": "0&"}

session.post(burp0_url, headers=burp0_headers, data=burp0_data)

```

### JSON Response

**German Radars / Traffic Light Information**:

```json
{"code": 0,"url": "https://radarbot-db.s3-accelerate.amazonaws.com/radars_v3/radars_de.db"}
```

Do Note that there is also a `radars_v2` active url for all of the countries, and that may
use a different encrpytion standard for comaptiability w/ older systems.


## Accessing the RadarBot24 Database

The RadarBot24 Database is Encrypted, Meaning that the Decryption Mechanism is coded inside the the .apk file. Downloaded Databases are stored inside the `files/` directory in `/data/data` of the `com.vialsoft.radarbot_free` application.

### My Recommendations

1. We need to Hire a Android Application Reverse Engineering Professional to Proceed Forward
to see how the app is decrypting the database.


### Database Content

```binary
TUKAKAB?@I?B@BCKGAE@KcYGHIIKEB>GCAIEIKG>GCCBEEKVKG@KR0BACKDKAKAIFK@GI@@KEA>I@DACAKH>CEHDDHKVKG@Kguƒ„‚y~w<0R0FAKDKAKBI@K@GIDGKDG>FFIIAFKI>AFC@D@KVKF@KcsxÓ´~Š|ur‚ÓÌs{u<0R0CC~KDKAKGHK@GIDHKDG>FFIIFIKI>AFBHD@KVKF@KcsxÓ´~Š|ur‚ÓÌs{u<0R0CC~KDKAKBEIK@GIG@KDG>HIAHFEKI>C@GFFGKVKE@K\0B@AKDKBKBA@K@GIGAKDG>IABB@GKI>B@BEDDKVKE@Kc„s{qsxu‚0c„‚qÓ¯u<0\0AIDKDKBKB@BK@GIHFKEA>BDI@FFKF>B@EDFGKVKE@KRyƒxuy}u‚0c„‚qÓ¯u<0\0CGCKDKBKB@CK@H@@FKDH>ABGHEFKI>FEAGA@KVKE@Ke„„u~‡uy|u‚0c„‚qÓ¯u<
```

This file is present w/ an `info.json` containing the starting data from where we determined 
the new country.

```json
{
    "Globals": {
        "start_lat": 50.1099011,
        "start_long": 8.6757933
    }
}
```



## Report By Muneeb A. (C) 2023

Hire Me on Upwork (https://www.upwork.com/freelancers/muneeba51)
