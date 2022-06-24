# 3D Secure

### Architecture

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-19-14-35-02-image.png)

<img title="" src="file:///C:/Users/duyng/AppData/Roaming/marktext/images/2022-06-19-00-14-52-image.png" alt="" width="691" data-align="center">

<img title="" src="file:///C:/Users/duyng/AppData/Roaming/marktext/images/2022-06-19-00-16-13-image.png" alt="" width="683" data-align="center">

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-19-00-18-40-image.png)

- **<u>Gateway SSL</u>**:
  
  - Open SSL tunnel to backend service

- **<u>ACS handler cluster</u>**:
  
  - Handle Authentication request:
    
    - **Areq/Ares**: 
      
      - Check enrollment status, check card existed, exipire date, aggregate common info such as authentication method. 
      
      - After all info is valid, checking rule to determine the outcome such as require OTP_SMS or frictionless.
      
      - The rule is configure on oracle database, and then cache to redis, ACS server call procedure to redis.
    
    - **Creq/Cres**:
      
      - **Creq**: request URL of front end page with session ID
        
        - Validate OTP
      
      - **Cres**: return to Browser succesful service
    
    - **RReq/RRes**
