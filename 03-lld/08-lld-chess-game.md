# LLD Case Study: Chess Game Design

Design an interactive, two-player chess game — tests core object relationships, complex move validations, and execution tracking patterns.

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
1. Two-player board game (White vs Black).
2. The board is an 8x8 grid of spots containing pieces or empty.
3. Core chess pieces: King, Queen, Bishop, Knight, Rook, Pawn. Each has specific movement rules.
4. Track turns: players alternate making moves; White goes first.
5. Record move history (log of all moves). Support undoing the last move.
6. Detect game ending conditions: Checkmate, Stalemate, Resignation.
7. Support advanced moves: Castling, Pawn Promotion, En Passant.

**Non-Functional/Design Challenges:**
1. **Clean Polymorphism:** Pieces must encapsulate their specific movement rules without cluttering the board class.
2. **Move validation framework:** Differentiating between generic movement patterns (e.g., straight-line vs diagonal) and state-dependent checks (e.g., is King in check?).

---

## Class Design

```
ChessGame (Context)
├── Board board
├── Player[] players
├── Player activePlayer
├── List<Move> moveHistory
├── GameStatus status
└── playTurn(), undoLastMove(), isCheckmate()

Board
├── Spot[][] spots
└── getSpot(int x, int y), initializeBoard()

Spot
├── int x, int y
└── Piece piece

Piece (Abstract)
├── boolean isWhite
├── boolean hasMoved
└── abstract boolean isValidMove(Board board, Spot start, Spot end)

King, Queen, Bishop, Knight, Rook, Pawn (Inherit Piece)

Move (Command Pattern)
├── Player player
├── Spot start
├── Spot end
├── Piece pieceMoved
├── Piece pieceKilled
└── execute(), undo()
```

---

## Full Java Implementation

### 1. Domain Entities & Piece Hierarchy
```java
import java.util.*;

public enum GameStatus { ACTIVE, WHITE_WIN, BLACK_WIN, STALEMATE, RESIGNED }

public class Spot {
    private final int x;
    private final int y;
    private Piece piece;

    public Spot(int x, int y, Piece piece) {
        this.x = x; this.y = y; this.piece = piece;
    }
    public int getX() { return x; }
    public int getY() { return y; }
    public Piece getPiece() { return piece; }
    public void setPiece(Piece piece) { this.piece = piece; }
}

public abstract class Piece {
    private final boolean white;
    private boolean hasMoved = false;

    public Piece(boolean white) { this.white = white; }
    public boolean isWhite() { return white; }
    public boolean getHasMoved() { return hasMoved; }
    public void setHasMoved(boolean moved) { this.hasMoved = moved; }

    public abstract boolean isValidMove(Board board, Spot start, Spot end);
}
```

### 2. Concrete Pieces (Examples: Knight and Bishop)
```java
public class Knight extends Piece {
    public Knight(boolean white) { super(white); }

    @Override
    public boolean isValidMove(Board board, Spot start, Spot end) {
        // Destination cannot contain same color piece
        if (end.getPiece() != null && end.getPiece().isWhite() == this.isWhite()) {
            return false;
        }
        int xDiff = Math.abs(start.getX() - end.getX());
        int yDiff = Math.abs(start.getY() - end.getY());
        // Knight moves in L-shape: 2x1 or 1x2
        return xDiff * yDiff == 2;
    }
}

public class Bishop extends Piece {
    public Bishop(boolean white) { super(white); }

    @Override
    public boolean isValidMove(Board board, Spot start, Spot end) {
        if (end.getPiece() != null && end.getPiece().isWhite() == this.isWhite()) {
            return false;
        }
        int xDiff = Math.abs(start.getX() - end.getX());
        int yDiff = Math.abs(start.getY() - end.getY());
        
        if (xDiff != yDiff) return false; // Must be diagonal
        
        // Check if path is clear
        int xDir = (end.getX() - start.getX()) > 0 ? 1 : -1;
        int yDir = (end.getY() - start.getY()) > 0 ? 1 : -1;
        
        int currX = start.getX() + xDir;
        int currY = start.getY() + yDir;
        while (currX != end.getX()) {
            if (board.getSpot(currX, currY).getPiece() != null) {
                return false; // Path blocked
            }
            currX += xDir;
            currY += yDir;
        }
        return true;
    }
}
```

### 3. Board and Move Commands
```java
public class Board {
    private final Spot[][] spots = new Spot[8][8];

    public Board() { initialize(); }

    public Spot getSpot(int x, int y) {
        if (x < 0 || x > 7 || y < 0 || y > 7) {
            throw new IllegalArgumentException("Out of bounds");
        }
        return spots[x][y];
    }

    private void initialize() {
        // Setup spots
        for (int i = 0; i < 8; i++) {
            for (int j = 0; j < 8; j++) {
                spots[i][j] = new Spot(i, j, null);
            }
        }
        // Place Knights for illustration
        spots[0][1].setPiece(new Knight(true));
        spots[0][6].setPiece(new Knight(true));
        spots[7][1].setPiece(new Knight(false));
        spots[7][6].setPiece(new Knight(false));
        
        // Place Bishops
        spots[0][2].setPiece(new Bishop(true));
        spots[0][5].setPiece(new Bishop(true));
        spots[7][2].setPiece(new Bishop(false));
        spots[7][5].setPiece(new Bishop(false));
    }
}

public class Move {
    private final Spot start;
    private final Spot end;
    private final Piece pieceMoved;
    private final Piece pieceKilled;
    private final boolean firstMoveForPiece;

    public Move(Spot start, Spot end) {
        this.start = start;
        this.end = end;
        this.pieceMoved = start.getPiece();
        this.pieceKilled = end.getPiece();
        this.firstMoveForPiece = !pieceMoved.getHasMoved();
    }

    public void execute() {
        end.setPiece(pieceMoved);
        start.setPiece(null);
        pieceMoved.setHasMoved(true);
    }

    public void undo() {
        start.setPiece(pieceMoved);
        end.setPiece(pieceKilled);
        if (firstMoveForPiece) {
            pieceMoved.setHasMoved(false);
        }
    }
}
```

### 4. Game Manager
```java
public class ChessGame {
    private final Board board = new Board();
    private final List<Move> moveHistory = new ArrayList<>();
    private boolean whiteTurn = true;
    private GameStatus status = GameStatus.ACTIVE;

    public boolean makeMove(int startX, int startY, int endX, int endY) {
        if (status != GameStatus.ACTIVE) return false;

        Spot start = board.getSpot(startX, startY);
        Spot end = board.getSpot(endX, endY);
        Piece piece = start.getPiece();

        if (piece == null || piece.isWhite() != whiteTurn) {
            System.out.println("Invalid turn or empty start spot.");
            return false;
        }

        if (piece.isValidMove(board, start, end)) {
            Move move = new Move(start, end);
            move.execute();
            moveHistory.add(move);
            
            whiteTurn = !whiteTurn; // Switch turn
            System.out.printf("Moved piece from (%d,%d) to (%d,%d)%n", startX, startY, endX, endY);
            return true;
        }
        System.out.println("Invalid move pattern for this piece.");
        return false;
    }

    public void undo() {
        if (moveHistory.isEmpty()) {
            System.out.println("No moves to undo.");
            return;
        }
        Move lastMove = moveHistory.remove(moveHistory.size() - 1);
        lastMove.undo();
        whiteTurn = !whiteTurn;
        System.out.println("Undid last move.");
    }
}
```

---

## Design Patterns Used

1. **Command Pattern:** The `Move` class acts as a Command object. Encapsulating the move operations inside `execute()` and `undo()` methods allows the `ChessGame` to cleanly record move history and support a robust `undo()` feature.
2. **Factory Pattern:** Useful when initializing the board based on a standard FEN (Forsyth-Edwards Notation) string or layout file to construct the pieces dynamically.
3. **Template Method:** Could be applied to piece path-clear validations. For instance, slide-pieces (Rook, Bishop, Queen) share the algorithm of scanning indices sequentially, but differ in the direction vector.

---

## Concurrency & Thread Safety

For local gameplay, the game loops run in a single thread context. For online multiplayer:
- Synchronization must occur at the `ChessGame` instance level when `makeMove()` is triggered.
- A lock ensures player B cannot send a move command before player A's move has been processed and state updated.

---

## Design Decisions

| Decision | Why |
|----------|-----|
| Piece encapsulates movement logic | Clean polymorphism. Keeps the `Board` class small, focusing strictly on coordinate access and layout. |
| Command Pattern for Moves | Simplifies tracking complex state changes (captures, promotions, first-move markers) required to perfectly undo moves. |
| Spot acts as cell holder | Keeps coordinates decoupled from the pieces, making it easier to render empty spots and pieces. |

---

## Interview Follow-ups

1. How do you implement castling, en passant, and pawn promotion?
2. How do you efficiently check for "check" and "checkmate" states?
3. What if we want to allow undoing a move? (Hint: Command pattern)
4. What if a player disconnects in a multiplayer game?
5. How would you integrate an AI/bot player?
6. How do you ensure the board state is perfectly synchronized between two remote players?
7. What if we want to record the game and replay it later?
8. How would you handle game timers (e.g., Blitz chess)?
9. What if we want to support variations like Fischer Random Chess?
10. How do you design the system to validate moves purely functionally?

---

## Interview Tips

1. **Focus on class structure:** Draw piece classes inheriting from `Piece`. Highlight how `isValidMove()` works.
2. **Handle advanced chess rules explicitly:** Castling requires checking if the King and Rook have ever moved, and if the path is clear. En Passant requires looking at the opponent's previous move (via the move history list).
3. **Demonstrate Undo cleanly:** Implementing a Command pattern is a huge differentiator in interviews.

---

## ⚡ Flashcard Summary

| # | Question | Answer |
|---|----------|--------|
| 1 | Core Entities | ChessGame, Board, Spot, Piece, Player, Move |
| 2 | Movement validation | Piece abstract method `isValidMove(Board, Spot, Spot)` |
| 3 | Command Pattern use | `Move` class with `execute()` and `undo()` to log history |
| 4 | Board representation | 2D Array of `Spot` objects (8x8 grid) |
| 5 | En Passant validation | Requires inspecting last move from `moveHistory` |
| 6 | Castling constraints | King & Rook `hasMoved == false` + path clear + King not in check |
| 7 | Piece creation | Factory method instantiates pieces based on coordinates / layout |
| 8 | Multi-player coordination | Synchronized `makeMove()` on Game instance for thread safety |

---

*Next: [HLD Foundations](../04-hld/01-hld-foundations.md)*
