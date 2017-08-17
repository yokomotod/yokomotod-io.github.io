+++
title = "tsc --watch --skipLibCheck"
date = 2017-08-18T03:07:19+09:00
tags = ["typescript"]
draft = false
+++


```sh
tsc --watch --skipLibCheck
```

good speed.

normal full compile
```
Files:           464
Lines:         97878
Nodes:        431416
Identifiers:  142914
Symbols:      108604
Types:         37006
Memory used: 230040K
I/O read:      0.05s
I/O write:     1.35s
Parse time:    1.27s
Bind time:     0.59s
Check time:    4.40s
Emit time:     3.50s
Total time:    9.77s
```

full compile + skipLibCheck
```
Files:           464
Lines:         97878
Nodes:        431416
Identifiers:  142914
Symbols:       92415
Types:         17081
Memory used: 199322K
I/O read:      0.04s
I/O write:     0.80s
Parse time:    1.31s
Bind time:     0.60s
Check time:    2.14s
Emit time:     2.20s
Total time:    6.25s
```

incremental compile (2nd and later with --watch) + skipLibCheck
```
03:41:42 - File change detected. Starting incremental compilation...
Files:           464
Lines:         97879
Nodes:        431416
Identifiers:  142914
Symbols:       92415
Types:         17081
Memory used: 264557K
I/O read:      0.00s
I/O write:     0.06s
Parse time:    0.21s
Bind time:     0.00s
Check time:    2.06s
Emit time:     1.12s
Total time:    3.40s
03:41:45 - Compilation complete. Watching for file changes.
```

nice.

I knew `skipLibCheck` but i thought it was too risky to skip, however, now I agreed that it's enough safe that use this option only on local and CI build will have full check.

And I didn't know that the tsc will transpile incrementaly with `watch` !
