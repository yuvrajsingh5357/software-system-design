# LLD Case Study: ATM Machine Design

Design an ATM (Automated Teller Machine) system — tests state transitions, hardware integration modeling, and cash dispensing algorithms.

## Table of Contents
- [Requirements Analysis](#requirements-analysis)
- [Class Design](#class-design)
- [Full Java Implementation](#full-java-implementation)
- [Design Patterns Used](#design-patterns-used)
- [Concurrency & Thread Safety](#concurrency--thread-safety)
- [Design Decisions](#design-decisions)
- [Interview Tips](#interview-tips)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## Interview Flow

1. Clarify requirements
2. Identify actors
3. Identify core entities
4. Identify relationships
5. Identify responsibilities
6. Define interfaces
7. Choose abstractions
8. Select patterns only where justified
9. Handle concurrency
10. Discuss extensibility
11. Write code
12. Test
13. Discuss trade-offs

---

## Requirements Analysis

**Functional Requirements:**
1. Insert card and authenticate via PIN.
2. Select transaction: Check Balance, Withdraw Cash, Deposit Cash.
3. Dispense cash in correct denominations (e.g., $100, $50, $20) based on availability.
4. Update account balances upon withdrawal or deposit.
5. Print transaction receipt.
6. Cancel transaction at any point, ejecting the card.

**Non-Functional Requirements:**
1. **Consistency:** Withdrawals must be atomic. If cash dispensing fails, the bank transaction must rollback.
2. **State Control:** ATM must reject operations if not in the correct state (e.g., cannot withdraw cash without authenticating).

**Core Complexity:**
- Managing the physical-to-logical state transitions of the machine.
- Choosing and dispensing notes (e.g., Chain of Responsibility).

---

## Class Design

```
ATM (Singleton/Context)
├── ATMState (Current State)
├── CashDispenser (Dispensing Chain)
├── Card (Inserted Card)
├── double availableCash
└── insertCard(), enterPin(), withdrawCash(), ejectCard()

ATMState (Interface)
└── insertCard(), enterPin(), selectTransaction(), withdraw(), ejectCard()

IdleState, HasCardState, PinEnteredState, NoCashState (Implement ATMState)

CashDispenser (Abstract Class / Chain Element)
├── CashDispenser nextDispenser
└── dispense(long amount)

100Dispenser, 50Dispenser, 20Dispenser (Inherit CashDispenser)

BankService
└── authenticateCard(), updateBalance()
```

---

## Full Java Implementation

### 1. State Pattern Interfaces and Classes
```java
import java.util.*;

public interface ATMState {
    void insertCard(ATM atm, Card card);
    void enterPin(ATM atm, int pin);
    void withdrawCash(ATM atm, double amount);
    void ejectCard(ATM atm);
}

public class IdleState implements ATMState {
    @Override
    public void insertCard(ATM atm, Card card) {
        atm.setCard(card);
        atm.setCurrentState(new HasCardState());
        System.out.println("Card inserted. Please enter PIN.");
    }
    @Override public void enterPin(ATM atm, int pin) { System.out.println("Insert card first."); }
    @Override public void withdrawCash(ATM atm, double amount) { System.out.println("Insert card first."); }
    @Override public void ejectCard(ATM atm) { System.out.println("No card to eject."); }
}

public class HasCardState implements ATMState {
    @Override public void insertCard(ATM atm, Card card) { System.out.println("Card already inserted."); }
    
    @Override
    public void enterPin(ATM atm, int pin) {
        if (atm.getBankService().authenticatePin(atm.getCard(), pin)) {
            atm.setCurrentState(new PinEnteredState());
            System.out.println("PIN accepted. Choose transaction.");
        } else {
            System.out.println("Invalid PIN. Card ejected.");
            ejectCard(atm);
        }
    }
    
    @Override public void withdrawCash(ATM atm, double amount) { System.out.println("Enter PIN first."); }
    
    @Override
    public void ejectCard(ATM atm) {
        atm.setCard(null);
        atm.setCurrentState(new IdleState());
        System.out.println("Card ejected.");
    }
}

public class PinEnteredState implements ATMState {
    @Override public void insertCard(ATM atm, Card card) { System.out.println("Card already inserted."); }
    @Override public void enterPin(ATM atm, int pin) { System.out.println("PIN already entered."); }
    
    @Override
    public void withdrawCash(ATM atm, double amount) {
        if (atm.getAvailableCash() < amount) {
            System.out.println("ATM out of cash.");
            ejectCard(atm);
            return;
        }
        
        Card card = atm.getCard();
        double userBalance = atm.getBankService().getBalance(card);
        
        if (userBalance >= amount) {
            // Dispense cash using Dispenser Chain
            boolean dispensed = atm.getDispenserChain().dispense((long) amount);
            if (dispensed) {
                atm.getBankService().deductAmount(card, amount);
                atm.deductCash(amount);
                System.out.printf("Dispensed: $%.2f. Account updated.%n", amount);
            } else {
                System.out.println("ATM cannot dispense exact denominations. Transaction cancelled.");
            }
        } else {
            System.out.println("Insufficient funds in account.");
        }
        ejectCard(atm);
    }
    
    @Override
    public void ejectCard(ATM atm) {
        atm.setCard(null);
        atm.setCurrentState(new IdleState());
        System.out.println("Card ejected.");
    }
}
```

### 2. Cash Dispenser (Chain of Responsibility)
```java
public abstract class NoteDispenser {
    protected NoteDispenser nextDispenser;

    public void setNextDispenser(NoteDispenser next) {
        this.nextDispenser = next;
    }

    public abstract boolean dispense(long amount);
}

public class Dollar100Dispenser extends NoteDispenser {
    @Override
    public boolean dispense(long amount) {
        long numNotes = amount / 100;
        long remainder = amount % 100;
        
        if (numNotes > 0) {
            System.out.printf("Dispensing %d notes of $100%n", numNotes);
        }
        
        if (remainder != 0) {
            if (nextDispenser != null) {
                return nextDispenser.dispense(remainder);
            }
            return false; // Chain ended, cannot dispense
        }
        return true;
    }
}

public class Dollar50Dispenser extends NoteDispenser {
    @Override
    public boolean dispense(long amount) {
        long numNotes = amount / 50;
        long remainder = amount % 50;
        
        if (numNotes > 0) {
            System.out.printf("Dispensing %d notes of $50%n", numNotes);
        }
        
        if (remainder != 0) {
            if (nextDispenser != null) {
                return nextDispenser.dispense(remainder);
            }
            return false;
        }
        return true;
    }
}

public class Dollar20Dispenser extends NoteDispenser {
    @Override
    public boolean dispense(long amount) {
        long numNotes = amount / 20;
        long remainder = amount % 20;
        
        if (numNotes > 0) {
            System.out.printf("Dispensing %d notes of $20%n", numNotes);
        }
        
        if (remainder != 0) {
            if (nextDispenser != null) {
                return nextDispenser.dispense(remainder);
            }
            return false;
        }
        return true;
    }
}
```

### 3. Context (ATM Class)
```java
public class ATM {
    private ATMState currentState;
    private double availableCash;
    private Card card;
    private final NoteDispenser dispenserChain;
    private final BankService bankService;

    public ATM(double initialCash, BankService bankService) {
        this.availableCash = initialCash;
        this.bankService = bankService;
        this.currentState = new IdleState();
        
        // Setup Dispenser Chain
        NoteDispenser d100 = new Dollar100Dispenser();
        NoteDispenser d50 = new Dollar50Dispenser();
        NoteDispenser d20 = new Dollar20Dispenser();
        d100.setNextDispenser(d50);
        d50.setNextDispenser(d20);
        this.dispenserChain = d100;
    }

    public void insertCard(Card card) { currentState.insertCard(this, card); }
    public void enterPin(int pin) { currentState.enterPin(this, pin); }
    public void withdrawCash(double amount) { currentState.withdrawCash(this, amount); }
    public void ejectCard() { currentState.ejectCard(this); }

    // Getters and Setters
    public ATMState getCurrentState() { return currentState; }
    public void setCurrentState(ATMState state) { this.currentState = state; }
    public Card getCard() { return card; }
    public void setCard(Card card) { this.card = card; }
    public double getAvailableCash() { return availableCash; }
    public void deductCash(double amount) { this.availableCash -= amount; }
    public NoteDispenser getDispenserChain() { return dispenserChain; }
    public BankService getBankService() { return bankService; }
}

public class Card {
    private final String cardNumber;
    public Card(String number) { this.cardNumber = number; }
    public String getCardNumber() { return cardNumber; }
}

public class BankService {
    public boolean authenticatePin(Card card, int pin) { return pin == 1234; }
    public double getBalance(Card card) { return 1000.0; }
    public void deductAmount(Card card, double amount) {
        System.out.printf("Deducted $%.2f from card account %s%n", amount, card.getCardNumber());
    }
}
```

---

## Design Patterns Used

1. **State Pattern:** ATM transitions through distinct states: `IdleState`, `HasCardState`, `PinEnteredState`. Using this pattern avoids messy `if-else` blocks checks in every method to verify if card is inserted, PIN is checked, or ATM has cash.
2. **Chain of Responsibility Pattern:** Used in `NoteDispenser` chain (100 -> 50 -> 20) to dispense cash. It decouples the request for dispensing from the specific note types.
3. **Singleton Pattern:** Typically applied to the `ATM` class since one physical machine has exactly one instance.

---

## Concurrency & Thread Safety

- **Atomic Transactions:** In production, the step of checking account balance, checking ATM cash, updating DB, and dispensing cash must be wrapped in a single database transaction (`@Transactional` with serializable isolation level or pessimistic locking) to prevent overdrafts.
- **Physical ATM Lock:** A physical ATM processes requests sequentially. Thus, access to the `withdrawCash()` method can be globally synchronized for the physical ATM runtime thread.

---

## Design Decisions

| Decision | Why |
|----------|-----|
| State Pattern for ATM States | Clear transitions. Ensures authentication is verified before any state modification. |
| Chain of Responsibility for Notes | Extremely easy to plug-in new currency note dispensers (e.g., $10 or $5 notes) without altering existing class definitions (supports Open-Closed Principle). |
| Decoupled `BankService` | ATM machine acts as a client to a banking core, keeping hardware logic separate from ledger operations. |

---

## Interview Follow-ups

1. What if the ATM loses network connection right after dispensing cash?
2. What if two people withdraw from the same account at different ATMs simultaneously?
3. What if the cash dispenser gets jammed?
4. How would you add support for depositing checks?
5. What if the ATM runs out of a specific denomination of notes?
6. How do you ensure transaction atomicity (all steps succeed or none do)?
7. What if the card is retained by the machine?
8. How would you handle multi-currency withdrawals?
9. What if a new type of transaction (e.g., bill payment) is added?
10. How do you securely handle PIN validation without logging it?

---

## Interview Tips

1. **Draw the State Diagram:** Immediately draw a state machine during the interview. Show states like `Idle`, `HasCard`, `PinEntered`, `NoCash`.
2. **Show Chain of Responsibility:** Explain why chain of responsibility is better than hardcoding modulo division (makes it easy to add new denominations).
3. **Highlight hardware boundaries:** Mention how hardware failure (e.g., cash jam) is handled. "If the dispenser fails, we reverse the ledger transaction in the bank database."

---

## ⚡ Flashcard Summary

| # | Question | Answer |
|---|----------|--------|
| 1 | Key ATM States | Idle, HasCard, PinEntered, OutOfCash |
| 2 | State Pattern utility | Encapsulates state-specific behavior, avoiding nested conditional checks |
| 3 | Note dispensing pattern | Chain of Responsibility (`NoteDispenser` chain) |
| 4 | Note dispensing ordering | Sort note size descending (100 → 50 → 20) to use largest first |
| 5 | Denomination failure | Chain returns false if amount cannot be fully divided; roll back transaction |
| 6 | Database consistency | Ledger deduction occurs *after* successful dispenser confirmation |
| 7 | Hardware exceptions | ATM catches dispenser jam → trigger rollback API on BankService |
| 8 | SOLID: Open-Closed | New dispensers added by extending `NoteDispenser` and linking them |

---

*Next: [LLD Case Study — Chess Game](08-lld-chess-game.md)*
