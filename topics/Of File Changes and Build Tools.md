Of File Changes and Build Tools
===

March 8, 2016

I was working in a project where they made use of a build tool to compress all the static files into a single file. While it was a decent idea, the execution was not. Usually they will be coupled with Grunt or Gulp (or the latest 3.0 web trend) to detect the file changes, and execute the build tool. Surprise, they don't.

Additionally, due to security concerns, I cannot install these tools as well. After manually calling the build tool for every single small change in CSS, I was so done with it, and looked for ways to automate this. If I ever fail, there's still `while true { call_build_tool }`.

I remembered during my Arch Linux days, I had to execute `ls -R` to help seed the random generator. That would show me all the files, and with `--full-time`, I will know if a file was modified. I thought of comparing this whole output, but a hash would suffice. With a short time, I bashed together something like so.

```sh
#!/bin/sh

# create a lock file
current_dir=$( cd "$(dirname "${BASH_SOURCE}")" ; pwd -P )
lock_file="$current_dir/watch.lock"
touch "$lock_file"

# get the other paths
project_path="..."

# keep in loop
while true; do
    current_hash=$(ls -lR --full-time "$project_path" | md5sum | cut -d ' ' -f 1)
    previous_hash=$(cat "$lock_file")
    
    # compare the hashes
    if [[ "$current_hash" != "$previous_hash" ]]; then
        echo -n "$current_hash" > "$lock_file"
        
        # call build tool
    fi
    
    # rest in peace
    sleep 1
done
```

By now, some devout Bash scripters would be cursing this piece of trash. Let me establish a couple more restrictions.

- I'm on Windows (gotcha).
- The GNU tools are provided by msysgit.
- msysgit version was old (and cannot be updated).
- `md5sum` is the only hasher in `/bin/`.

With that in mind, I am impressed with this masterpiece. I booted it and it worked. However, it was painfully slow, at about one to two seconds for each time it calls `ls -lR` (yes, the sleep time was not counted in). Only after this did I decide to consult with the great scholars of Google and Stackoverflow.

[I got](http://stackoverflow.com/questions/2972765/linux-script-that-monitors-file-changes-within-folders-like-autospec-does) [to know](http://superuser.com/questions/181517/how-to-execute-a-command-whenever-a-file-changes) [about `inotify`](http://unix.stackexchange.com/questions/24952/script-to-monitor-folder-for-new-files), but alas, it was not available on the system. However, I found a [different approach](http://stackoverflow.com/questions/4561895/how-to-recursively-find-the-latest-modified-file-in-a-directory), where somebody used `find` instead of `ls`. I changed the relevant parts to use `find "$project_path" -type f -printf '%T@ %p\n'` instead and found out that it was a lot faster. So its time to use `time` (which does not support any additional parameter, because its so old).

|  | `find` | `ls` |
| --- | --- | --- |
| real | 0m0.780s | 0m1.903s |
| user | 0m0.076s | 0m0.202s |
| sys  | 0m0.543s | 0m0.792s |

So `find` is really faster than `ls`. I doubt I can squeeze any more from this (unless you want to further [scope the files with `-name`](http://superuser.com/questions/126290/find-files-filtered-by-multiple-extensions)), so I wrapped it up, and started enjoying hands-free building process.

```sh
#!/bin/sh

# create a lock file
current_dir=$( cd "$(dirname "${BASH_SOURCE}")" ; pwd -P )
lock_file="$current_dir/watch.lock"
touch "$lock_file"

# get the other paths
project_path="..."

# just in case you think i's dead
echo -e "\nI'm listening alright, just get on with work, and CTRL + C when you're done."

# keep in loop
while true; do
    current_hash=$(find "$project_path" -type f -printf '%T@ %p\n' | md5sum | cut -d ' ' -f 1)
    previous_hash=$(cat "$lock_file")
    
    # compare the hashes
    if [[ "$current_hash" != "$previous_hash" ]]; then
        echo -n "$current_hash" > "$lock_file"
        
        # call build tool
        
        timestamp=$(date +%Y-%m-%dT%H:%M:%S%z)
        echo "$timestamp executed build tool"
    fi
    
    # rest in peace
    sleep 1
done
```

---

#### References
- http://stackoverflow.com/questions/2972765/linux-script-that-monitors-file-changes-within-folders-like-autospec-does
- http://superuser.com/questions/181517/how-to-execute-a-command-whenever-a-file-changes
- http://unix.stackexchange.com/questions/24952/script-to-monitor-folder-for-new-files
- http://stackoverflow.com/questions/4561895/how-to-recursively-find-the-latest-modified-file-in-a-directory
- http://superuser.com/questions/126290/find-files-filtered-by-multiple-extensions
