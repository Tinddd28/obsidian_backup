
![[Pasted image 20240901215506.png]]

4 стандартных уровня изоляции 
American National Standarts Institute - ANSI

Low
1) "Read uncommitted": can see data written by uncommitted transaction -> dirty read
2) "Read committed": only see data written by committed transaction -> dirty read is impossible
3) "Repeatable read": same read query always returns same result
4) "Serializable": can arhieve same result if execut transactions serially un some order instead of concurrently
High