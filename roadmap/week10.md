
Week 10:

Implement JobQueues for asynchronous service worker lifecycle propagation.

[X] Implement Start Register

[X] Implement Schedule Job 

[X] Implement Register Job

[X] Implement Reject Job and Resolve Job

[ ] Implement Update Job

[ ] Implement Install Algorithm

[ ] Implement Clear Registration

[ ] Implement Unregister Algorithm


Other areas and notes to self:

* Check promises on equivalent jobs

* Update the origin trustworthy check algorithm.

* Implement remaining parts of update algorithm.

* A scope cannot be more generic then, the path of the script url, so in that case we should reject with a security error (in Update algorithm)
