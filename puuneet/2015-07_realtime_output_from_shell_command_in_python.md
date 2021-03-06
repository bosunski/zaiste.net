---
created_at: 2015-07-02
kind: article
publish: true
title: "Realtime output from a shell command in Python"
tags:
- python
- cli
- shell
---

Running a shell command in Python usually waits until the process is finished
and only then sends its entire output. In the following example it is
`process.communicate()` that blocks till given command is completed.

    import subprocess
    import shlex

    command = shlex.split("ping -c 5 google.com")
    process = subprocess.Popen(command, stdout=subprocess.PIPE)
    output, err = process.communicate()
    print output

`subprocess` module allows to capture realtime output as well - here's how:

    from subprocess import Popen, PIPE

    def run(command):
        process = Popen(command, stdout=PIPE, shell=True)
        while True:
            line = process.stdout.readline().rstrip()
            if not line:
                break
            yield line


    if __name__ == "__main__":
        for path in run("ping -c 5 google.com"):
            print path
