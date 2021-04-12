# Atomic Operations

An **atomic operation** is a sequence of one of more statements that is / appears to be indivisible i.e. they will always be executed without any potential for interrupts in the middle of execution.

**Warning**: not every single statement is atomic. For example:

```c
void withdraw(int account_no, int amount) {
  int balance = accounts[account_no];
  accounts[account_no] = balance - amount;
}
```

could be rewritten as:

```c
void withdraw(int account_no, int amount) {
  accounts[account_no] -= amount;
}
```

but it would **not** be atomic as `-=` is simply syntactic sugar for what was written before.

