## HW2: Defusing a binary bomb

## Setup:

1. Download the tarball for your architecture at the following links:

   X86 Windows/ Intel Mac/ Linux: [https://drive.google.com/file/d/1Xqg_iA2pQKIElTnBCEVArfQKeU_ZAin8/view?usp=sharing](https://drive.google.com/file/d/1Xqg_iA2pQKIElTnBCEVArfQKeU_ZAin8/view?usp=sharing)

   Apple Silicon Mac:[https://drive.google.com/file/d/12W3ZM2gmGiPreEz4iZgpLl6DI9ftmJC5/view?usp=sharing](https://drive.google.com/file/d/12W3ZM2gmGiPreEz4iZgpLl6DI9ftmJC5/view?usp=sharing)

2. Install Docker Desktop, if not already installed. See README for HW1 for instructions on that.

   Apple Silicon Mac: `bomblab-student-arm.tar` Image Tag: `bomblab-student:arm64`

   X86_64 Windows/Intel Mac/Linux: `bomblab-student-x86.tar` Image Tag: `bomblab-student:amd64`

3. Load the image in a terminal

   Apple Silicon Mac

   ```
   docker load -i bomblab-student-arm.tar
   ```

   X86 Windows/Intel Mac/Linux

   ```
   docker load -i bomblab-student-x86.tar
   ```

4. Fetch your bomb

   a. Create a working folder

   ```
   mkdir work
   ```

   b. Get the bomb. NOTE: Your GitHub username is case-sensitive to what you submitted on the Form. If you need confirmation, please let the Professor or TAs know.

   Apple Silicon Mac

   ```
   docker run --rm -v "$(pwd)/work:/work" bomblab-student:arm64 get-bomb <your-github-username>
   ```

   X86 Windows/Intel Mac/Linux

   ```
   docker run --rm -v "C:/path/to/work:/work" bomblab-student:amd64 get-bomb <your-github-username>
   ```

5. Run your bomb

   a. Start the container using your architecture's tag:

   Apple Silicon Mac

   ```
   docker run --rm -it -v "$(pwd)/work":/work bomblab-student:arm64
   ```

   Windows/Intel Mac/Linux

   ```
   docker run --rm -it -v "C:\path\to\work:/work" bomblab-student:amd64
   ```

   b. Inside the container, run:

   ```
   qemu-riscv64 -L /usr/riscv64-linux-gnu /work/bomb
   ```

   You'll see:

   ```
   Welcome to my fiendish little bomb. You have 6 phases with which to blow yourself up. Have a nice day!
   ```

   Note: The bomb then waits for Phase 1's input. A correct answer prints a "defused" message; a wrong answer prints BOOM!!! and exits.

   Do `CTRL+C` to exit the bomb program.

## Solving the bomb

Open two terminals and in both, navigate to your working directory. When you run `ls` in your working directory, you should see the `work/` directory you made in Step 4a above.

Debug by attaching gdb-multiarch to QEMU's gdb stub.

In Terminal 1: Start the container using your architecture's tag:

Apple Silicon Mac

```
docker run --rm -it -v "$(pwd)/work":/work bomblab-student:arm64
```

Windows/Intel Mac/Linux

```
docker run --rm -it -v "C:\path\to\work:/work" bomblab-student:amd64
```

Then, in Terminal 1:

```
qemu-riscv64 -g 1234 -L /usr/riscv64-linux-gnu /work/bomb
```

Next, in Terminal 2:

```
docker exec -it <container-id> bash
```

To get the container ID, go to another terminal and run:

```
docker ps
```

Find the row whose image is `bomblab-student:amd64` (or `bomblab-student:arm64` for Mac) and copy its CONTAINER ID. For example, we can see that the container ID is `90cac378fd2d`:

```
CONTAINER ID   IMAGE                   COMMAND       CREATED         STATUS        PORTS     NAMES
90cac378fd2d   bomblab-student:arm64   "/bin/bash"   2 seconds ago   Up 1 second             admiring_cray
```

Then, in Terminal 2:

```
gdb-multiarch /work/bomb
```

In GDB, run the following two commands:

```
(gdb) set architecture riscv:rv64
(gdb) target remote :1234

```

Then, you're ready to start "defusing" the bomb using gdb commands.

First, run

```
(gdb) break explode_bomb
```

This command says stop when execution reaches the `explode_bomb()` function, before the bomb completes its explosion sequence. This allows you to inspect the state of the program after an incorrect input instead of immediately losing the debugging session.

Then, run

```
(gdb) break main
(gdb) continue
```

GDB resumes execution and should stop at `main()`. When you later use `continue` and the bomb reaches an input prompt, switch to Terminal 1 and type the answer there. Do not type the bomb's answer at the `(gdb)` prompt.

Note: The `break explode_bomb` line is the important one: it pauses the program just before it would print `BOOM!!!` and exit, so you can test a wrong answer without having to restart. If you want to restart, you can exit gdb by running

```
(gdb) exit
```

Then start again at the top of this section [Solving the Bomb](#solving-the-bomb).

Note: While debugging, the bomb's input is read from the terminal where `qemu-riscv64` was launched, not from the gdb prompt. You can also run the bomb against a file of candidate answers:

```
qemu-riscv64 -L /usr/riscv64-linux-gnu /work/bomb < answers.txt
```
