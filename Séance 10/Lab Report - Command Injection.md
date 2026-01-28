# Lab Report - Command Injection


# Overview 
---
- **Difficulty**: All
- **Platform**: Linux
- **Link**: https://tryhackme.com/room/dvwa

# Exploitation 
---
- To begin with, we are tasked to enter an IP address. 
- After doing so, the application appears to perform ping tests against the provided IP address. 

- Given that, we assume that the system is performing the following operation:
```bash
ping [IP]
```

## Using semicolon to  perform command injection
---
- The classic technique is to use semicolon in order to close the initial command, then execute an arbitrary one (which we would provide). 
- The command executed on the target would look like (POC):
```bash
ping 1.1.1.1; whoami #We injected: 1.1.1.1; whoami
```

- This works on the `Low` difficulty. 

## Using Logical operators to perform command injection
---
- Servers may perform filtering and strip the semicolon from our request. 
- A first idea may be to attempt to use multiple semicolons: `;;`. This would work if the filtering is not recursive. 

- However, we can also use logical operators such as: `&`, `&&`, `|`, `||`. Here is the use case of each on of them: 
	- `cmd1 & cmd2` : Execute cmd1 in the background.
	- `cmd1 && cmd2`: Execute cmd2 only if cmd1 succeeds. 
	- `cmd1 | cmd 2`: Pipes the output from cmd1 into cmd2.
	- `cmd1 || cmd2`: Execute cmd2 only if cmd1 fails.

- Hence, the trick would be to either make the first command succeed or fail on purpose. 
- Therefore, we used the following payload (inducing a fail in the first command) (POC):
```bash
ping dum || id #We injected: dum || id
```
- Here, the ping command expect an IP address, not a string. 

- This works on the `Medium` difficulty.

## Using parsing logic to perform command injection
---
- Upon inspecting the source code for the `Hard` level, we found that special characters such as `&`, `||`,... are stripped:
```PHP
// Set blacklist    
$substitutions = array(        
'||' => '',        
'&'  => '',        
';'  => '',        
'| ' => '',        
'-'  => '',       
'$'  => '',        
'('  => '',        
')'  => '',        
'`'  => '',       
);
```

- The flaw here is that there is a whitespace in the pipe operator filtering: `|`
- Therefore, `|id` is not filtered out. 
- Moreover, `|id` is equivalent to `| id` as the filesystem parses the first one into the second one. 

- Finally, the following payload allows us to bypass the filtering on the `Hard` difficulty:
```bash
ping 1.1.1.1 |id #We injected: 1.1.1.1 |id
```


# Remediation Summary
---
- Instead of a blacklist, it is often better to operate filtering based on a whitelist. 


# Lessons Learned
---
- Logical operators use-case. 
- Parsing discrepancies. 
