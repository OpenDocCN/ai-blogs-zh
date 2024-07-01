<!--yml
category: 未分类
date: 2024-07-01 18:18:03
-->

# It’s just a longjmp to the left : ezyang’s blog

> 来源：[http://blog.ezyang.com/2010/11/its-just-a-longjmp-to-the-left/](http://blog.ezyang.com/2010/11/its-just-a-longjmp-to-the-left/)

*And then a signal to the ri-i-i-ight.*

One notable wart with readline is that if you `^C` during the prompt, nothing happens, and if you do a second `^C` (and weren’t buggering around with the signal handlers) your entire program unceremoniously terminates. That’s not very nice! Fortunately, `readline` appears to be one of the rare C libraries that actually put some work into making sure that you could longjmp out of a signal handler and not completely break the library’s internal state (they do this with liberal masking and unmasking, and their own signal handler which cleans up and then rethrows the signal).

So I decided I was going to see if I could patch up readline to `longjmp` out of the signal handler (signal provided by [yours truly](http://blog.ezyang.com/2010/09/towards-platform-agnostic-interruptibility/)) and give control back to Haskell. This monstrosity resulted.

```
static jmp_buf hs_readline_jmp_buf;
static struct sigaction old_sigpipe_action;

void hs_readline_sigaction (int signo, siginfo_t *info, void *data)
{
    sigaction(SIGALRM, &old_sigpipe_action, NULL);
    siglongjmp(hs_readline_jmp_buf, signo);
}

char *hs_readline (const char *s)
{
    struct sigaction action;
    int signo;
    sigset_t mask;
    memset(&action, 0, sizeof(struct sigaction));
    sigemptyset(&mask);
    sigaddset(&mask, SIGALRM);
    action.sa_sigaction = hs_readline_sigaction;
    action.sa_mask = mask;
    action.sa_flags = SA_RESETHAND;
    sigaction(SIGALRM, &action, &old_sigpipe_action);
    if (signo = sigsetjmp(hs_readline_jmp_buf, 1)) {
        return NULL;
    }
    char *r = readline(s);
    sigaction(SIGALRM, &old_sigpipe_action, NULL);
    return r;
}

```

It actually works pretty wonderfully, despite the somewhat circuitous route the signal takes: the SIGINT will first get handled by *readline’s* installed signal handler, which will clean up changes to the terminal and then rethrow it to GHC’s signal handler. GHC will tell the IO manager that a signal happened, and then go back to the innards of readline (who reinstates all changes in the terminal). Then, the IO manager reads out the signal, and sends a `ThreadKilled` exception, which then results in the RTS trying to interrupt the foreign call. The `SIGALRM` (actually, that’s a lie, the code that’s in GHC sends a `SIGPIPE`, but readline doesn’t think a `SIGPIPE` is a signal it should cleanup after, so I changed it—better suggestions welcome) hits readline’s signal handler again, we clean up the terminal, and then we hit our signal handler, which longjmps to a `return NULL` which will take us back to Haskell. And then the signal is caught and there is much rejoicing.

Unfortunately, almost all of that code is boilerplate and I can’t stick it in a nice Haskell combinator because the when Haskell is executing there’s no stack to speak of, and I bet a `setjmp` FFI call would make the RTS very confused. It’s also not reentrant although I doubt `readline` is reentrant either. And of course, nonlocal control transfer from a signal handler is something your Mom always told you not to do. So this approach probably doesn’t generalize. But it’s pretty amusing.