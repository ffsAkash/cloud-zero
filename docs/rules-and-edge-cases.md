# Rules and edge cases

Use this guide to turn chess rules into explicit program behavior and tests. The authoritative reading is the [FIDE Laws of Chess](https://handbook.fide.com/chapter/E012023), particularly articles 3, 5, and 9. This project initially targets standard chess, not Chess960.

## Three different questions

1. **Does a piece attack a square?** A geometric and occupancy question with special pawn behavior.
2. **Is this a legal move?** A movement, special-state, and king-safety question.
3. **Is the game over or a draw claim available?** A rules and sometimes history question.

Keep these operations conceptually separate. Standard perft explores legal moves without applying repetition or move-count draw adjudication.

## Attacked squares

Pawn attacks are diagonal even when the target square is empty. Pawn pushes do not constitute attacks. Knights jump. Sliding pieces stop at blockers. Kings attack adjacent squares even though moving adjacent to the opposing king would be illegal.

A piece can count as attacking a square even when moving there would expose its own king. Consequently, deriving attacks from the opponent's legal move list can produce incorrect king and castling decisions.

### Test prompts

- Does a pawn prevent an enemy king from entering an empty diagonal square?
- Does a pinned knight still attack its normal target squares?
- Does a blocker stop a rook from attacking squares beyond it?
- After a king capture, does removing the captured piece reveal a sliding attacker?

## Checks and ordinary legality

After a candidate move, check the king belonging to the player who moved. Side to move may already have switched. This detail is a frequent source of wrong-color tests.

A legal reply to check can capture the checking piece, block a sliding attack, or move the king, depending on the position. Double check permits only king moves. Never implement checkmate as capture of the king: mate occurs while both kings remain on the board.

### Test prompts

- A pinned rook moving off its pin line.
- A pinned rook moving along the pin line.
- A king capturing a piece protected by a newly exposed bishop.
- A single-check position with a blocking reply.
- A double-check position containing tempting non-king moves.

## Pawn movement

A single push requires an empty destination. A double push requires the starting rank and two empty squares. Captures use different destinations from pushes. Test both colors independently.

After a completed legal move, a pawn on its last rank is replaced by the selected promotion piece. Generate queen, rook, bishop, and knight choices for both quiet promotions and capture promotions. A promotion creates four distinct moves even if several produce similar evaluations.

### Test prompts

- A blocked intermediate square on a double push.
- A pawn already away from its starting rank.
- Capture promotions on the a-file and h-file.
- An underpromotion that avoids stalemate.
- A promotion that gives check along a file or diagonal.

## En passant

An en passant opportunity follows the relevant opposing two-square pawn advance and lasts for the immediate reply only. The moving pawn lands on the target square; the captured pawn is removed from a different square.

Because two occupied squares become empty, en passant can expose a sliding attack that ordinary destination-only reasoning misses. Apply the complete transition before checking the mover's king.

FEN convention can record an en passant target after a double push even when no capture is possible. Parsing that field does not mean a legal en passant move necessarily exists. Preserve or canonicalize metadata according to a documented policy.

### State checklist

- Clear the previous opportunity on the next move.
- Create a new target only for the relevant double pawn push.
- Check that the capturing pawn and captured pawn occupy the expected squares.
- Remove the pawn from its actual captured square.
- Reset the halfmove clock as for other pawn moves/captures.
- Restore everything on undo.

### Test prompts

- An ordinary legal en passant capture for each color.
- An opportunity that expires after a quiet reply.
- A capture that would expose the king along a rank.
- A capture that removes a checking pawn.
- A nominal FEN target with no adjacent capturing pawn.
- An adjacent pawn whose en passant capture is illegal because of king safety.

## Castling

For standard chess, verify all of the following:

1. The relevant right exists.
2. The king and rook are on the required starting squares.
3. Squares between them are empty.
4. The king is not currently in check.
5. The king's transit square is not attacked.
6. The king's destination is not attacked in the relevant resulting occupancy.

Only the king's route is subject to attack restrictions. A rook may be attacked, and queenside b-file squares need not be safe for the king, although the square between rook and king must still be empty. Use correct occupancy when evaluating the king's movement; a stale attack map can hide a discovered ray.

### Rights updates

- Any king move removes both rights for that color.
- Moving a rook from its original corner removes that side's corresponding right.
- Capturing a rook on its original corner removes that rook's corresponding right.
- Returning pieces to their starting squares does not restore rights.
- A promoted rook does not create rights.

Do not infer rights solely from piece placement. Imported contradictory metadata should follow your validation policy rather than allowing castling with a nonexistent rook.

### Test prompts

Create one fixture for every condition above, changing only that condition. Include attacked rooks, attacked b-file squares, occupied b-file squares, king-out-and-back histories, rook-out-and-back histories, and rook capture on an original corner.

## Move counters

The halfmove clock counts plies since the last pawn move or capture. Reset it for all pawn moves and all captures, including promotion and en passant. Ordinary king and rook moves, including castling, increment it despite changing rights.

The fullmove number starts at one and increments after Black's move. Both counters must restore exactly on undo. Castling rights becoming irreversible does not imply the halfmove clock resets.

## Checkmate and stalemate

When there are no legal moves:

- In check means checkmate.
- Not in check means stalemate.

Do not classify a position from material evaluation. A side can have overwhelming material and still be stalemated. Search must recognize these cases even at its ordinary depth boundary and inside quiescence.

## Repetition identity

The same piece layout is insufficient. Repetition depends on the same player to move, the same pieces on the same squares, and the same move possibilities relevant to castling and en passant.

For en passant, distinguish a legal capture opportunity from an irrelevant raw target. If the only adjacent pawn is pinned so the capture is illegal, raw target metadata must not create a false distinction in repetition identity.

History must come from moves actually supplied or played. A FEN plus no prior moves does not reveal earlier occurrences.

### Test prompts

- A knight shuffle that repeats the position.
- The same shuffle ending with the opposite side to move.
- A repeated layout after a rook moved and returned.
- Equal layouts with a genuinely legal en passant option in only one.
- Equal layouts differing only in an unusable en passant target.
- Undoing search moves without erasing the actual game history.

## Claimable and automatic draws

Under the referenced FIDE rules:

| Condition | Meaning |
| --- | --- |
| Third occurrence, including qualifying intended-move claims | A draw can be claimed under the specified conditions |
| Fifty moves by each side without pawn move/capture | A draw can be claimed under the specified conditions |
| Fifth occurrence | Automatic draw |
| Seventy-five moves by each side without pawn move/capture | Automatic draw; checkmate on the final move takes precedence |

The move-count thresholds correspond to 100 and 150 plies respectively. Follow article 9 for the exact claim procedure. A search engine commonly adopts simplifying policies for draw claims or cycles, but those policies should be named and documented separately from a strict game adjudicator.

If a search treats a twofold repetition along the current path as a draw to prevent cycles, that is a search policy. It does not establish a threefold claim in the actual game.

## Dead positions

A dead position is one where no legal sequence can result in checkmate. This is stronger than saying one side cannot force mate.

Start conservatively with obvious cases such as bare kings, king and bishop versus king, and king and knight versus king. Broader recognition requires care with material configurations and blocked positions. Avoid labelling every unfamiliar low-material position a draw.

A material checklist is an intentionally incomplete recognizer, not a complete implementation of the dead-position rule. Record that limitation in engine documentation when applicable.

## Rules review exercise

For every completed rule, write:

```text
Rule:
State required to apply it:
Smallest legal example:
Smallest illegal example:
State changed by making the move:
State restored by undo:
Interaction with attacks:
Interaction with history or counters:
```

If you cannot fill in a field, investigate it before moving on. Special-move bugs often originate in an omitted state update rather than an incorrect destination square.
