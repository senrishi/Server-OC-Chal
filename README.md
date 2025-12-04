# Server-OC-Chal (Prototype Pollution + SSRF w/ black listed input)
Chal link - [Here](https://chal1-a5w6.onrender.com)
### Solution + Explanation
1. Load the website, and play around with the Overclock button for a while.
2. For the first stage, increment the OC from 51-55x sequentially. Token is generated only for this approach
3. After this, OC to 56x, and a token will automatically be stored in the cookies.
4. Decoding it with [jwt.io](https://jwt.io), we get an endpoint that accepts a JSON payload. An example payload is included.
5. Using the example payload in a POST request to the endpoint throws an error about invalid permissions. The path mentioned in the payload is ```C:\\Windows\\Log\\systemRestore```. Opening this in a Windows machine requires Admin privileges. So a boolean property (isAdmin) must be added to the payload.
6. Adding the boolean property only throws an error again. Add the ```__proto__``` setter to the admin property to mutate the object
 Actual payload :
```
curl -X POST http://localhost:3000/logs \
     -H "Content-Type: application/json" \
     -d '{
           "Path": "C:\\Windows\\Log\\systemRestore",
           "__proto__": {
               "isAdmin": true
           }
         }'
```
7. This returns the first half of the payload which says - "Pctf{L!qU1d_H3L1um_"
----------------------------------------------------
9. The second part is kind of a gamble. The liquid helium is a hint. Checking robots.txt, it gives a CPU name (not exactly the way to use robots.txt ikik)
10. Continue to 76x on the OC, which displays a message and a Benchmark button. Checking the Network tab on the console, it makes a GET request to an endpoint within the server.
11. Adding this "internal URL" to the benchmark endpoint, an error pops up, stating that it could not connect to the server. Replacing localhost with an IPv6 loopback address (or) oct/decimal conversion of 127.0.0.1 (or) using 127.1, the first part is bypassed. However, internal is another challenge, since its a forbidden word. Double encoding either the entire word or any letter, this is bypassed as well.
 A valid request :
- ``` "http://localhost:3000/benchmark?url=http://127.1:3000/benchmark?%2569%256e%2574%2565%2572%256e%256DD1%256c=flag" ```
- %2569%256e%2574%2565%2572%256e%256DD1%256c = internal
- the above string is internal double encoded in hex
12. This brings the second half of the flag - $h0ulD_N0T_T0uch_$3rv3rs}. Need to be a bit more creative with this tho

### The main stuff
Axios and the recursiveMerge function hard carry this chal. Recursive merge adds any new properties input by the user to the existing object.
- But considering that the line ```Object.preventExtensions(sessionUser);``` explicitly prevents any new properties from adding
- The proto setter modifies the objects protoype to include an Admin property, which now satisfies the condition.

- For the SSRF bypass, the strings ```localhost``` and ```127.0.0.1``` are also explicitly blocked. So technically, any other address or string should work. But axios will accept only an alteration of localhost for server side request. So it automatically interprets the other encodings or forms of 127.0.0.1.
- The "internal" word obfuscation is like an extra challenge. The double encoded part is automatically decoded by express and axios, so it works.
