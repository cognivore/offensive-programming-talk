# Offensive Programming 
## in Elixir with Uptight Library

Jonn Mostovoy

Serokell.io & Doma.dev

---

# Let's talk about errors!

 - Recoverable (think `Result<E, O>`)
 - Irrecoverable  (think `panic!()`)

---

# Fault-tolerant

 - Recoverable (think `Result<E, O>`)
 - Supervised (think `throw-catch`)

---

# Fault-tolerant vs correctness

 - FT systems: survive the "chaos monkey"
 - Correct systems (?): evidence that "chaos monkey" won't happen

---

# Correctness and complex systems

 - Complexity causes emergence
 - You can't precisely / statically analyse it

---

# Offensive Programming
## Fault tolerance maximised

---

# Erlang's legacy

```bash
cat <<EOF | iex -S mix
File.read("flake.nix") |> elem(0)
EOF
```

---

# Erlang's legacy: minimise data flow

 - Atoms encode into ints
 - Lightest error tagging under distribution
 - For most use cases it's a non-issue
    - (we wouldn't have been using HTTP then)

---

# Erlang's legacy: nonsense

```
{:ok, {:error, "Something went wrong"}}
```

---

# Erlang's legacy: cardinality

```
{:error, "Something went boom"}
{:error, "Something went boom", %{original :data}}
{:error, %{original: :data}}
```

---

# Erlang's legacy: BOTH AT THE SAME TIME!

```
# Yes, this exists in the wild!
{:ok, %{error:
    %{reason: "something went boom",
      data: %{original: :data}}
}}
```

---

# Elixir: Exceptions!

 - Exceptions are verbose
 - Allow access to stacktrace

---

# Uptight

```bash
cat <<EOF | iex -S mix
File.read!("flake.nixx")
EOF
```

---

# Uptight

```bash
cat <<EOF | iex -S mix
alias Uptight.Result
:erlang.system_flag(:backtrace_depth, 1)
Result.new(fn -> File.read!("flake.nixx") end)
EOF
```

---

# Uptight: Functional

```bash
cat <<EOF | iex -S mix
use Witchcraft
alias Uptight.Result
import Witchcraft.Comonad

Result.new(fn -> 42 end)
~> (&(&1 + 1))
|> extract() == 43
EOF
```

---

# Uptight: Monadic

```bash
cat <<EOF | iex -S mix
use Witchcraft
alias Uptight.Result
import Witchcraft.Comonad
:erlang.system_flag(:backtrace_depth, 1)

Result.new(fn -> 5 = 2 + 2 end)
~> (&(&1 + 1))
~> (&(&1 + 1))
EOF
```

---

# Uptight: Budget-friendly

```bash
cat <<EOF | iex -S mix
alias Uptight.Result
import Uptight.Assertions
:erlang.system_flag(:backtrace_depth, 1)
Result.new(fn -> 
    assert match?(5, 2 + 2), "big brother is watching"
end).err.exception.message
EOF
```

---

# Links

 - [Uptight: Tighter typing and error-(un)-handling in Elixir](https://github.com/doma-engineering/uptight)
 - [J. Armstrong: Scalable, Fault tolerant, Upgradable Systems](http://armstrongonsoftware.blogspot.com/2007/07/scalable-fault-tolerant-upgradable.html)
 - [B. Zelenka: Exceptional: Freedom from `{:error}`s](https://medium.com/the-monad-nomad/exceptional-freedom-from-error-s-eaabfae25d72#.zgbne4gja)
 - [T. Neely: Error Handling in a Correctness-Critical Rust Project](http://sled.rs/errors.html)
