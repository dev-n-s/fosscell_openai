/**
 * Fold a character grid and resolve letter collisions.
 *
 * Each input cell may be a string (one letter) or an array of letters.
 * Piles are stored bottom-to-top; a fold flips an incoming pile and drops it
 * onto the target. Folds are "left", "right", "up", or "down".
 * Empty final cells are rendered as "_".
 *
 * @param {(string | string[])[][]} grid
 * @param {("left" | "right" | "up" | "down")[]} folds
 * @returns {string[][]} top-view grid, with piles rendered bottom-to-top
 */
function squeezeMatrix(grid, folds) {
  if (!Array.isArray(grid) || grid.length === 0 || !grid.every(Array.isArray)) {
    throw new TypeError("grid must be a non-empty rectangular array");
  }

  const width = grid[0].length;
  if (width === 0 || !grid.every((row) => row.length === width)) {
    throw new TypeError("grid must be a non-empty rectangular array");
  }

  // Internal pile order is bottom-to-top; the last element is the top letter.
  let board = grid.map((row) => row.map((cell) => {
    const pile = typeof cell === "string" ? [...cell] : [...cell];
    if (pile.some((letter) => typeof letter !== "string" || [...letter].length !== 1)) {
      throw new TypeError("each cell must contain single-character strings");
    }
    return pile;
  }));

  const isVowel = (letter) => /^[aeiou]$/i.test(letter);

  // Drop a flipped source pile onto a target. The source is reversed by the
  // physical flip. Each arriving letter contacts the current top of the pile.
  function drop(target, source) {
    const result = target.slice();
    for (const letter of source.slice().reverse()) {
      const top = result[result.length - 1];
      if (top === undefined) {
        result.push(letter);
      } else if (isVowel(top) !== isVowel(letter)) {
        result.pop(); // vowel and consonant both disappear
      } else if (top.toLowerCase() === letter.toLowerCase()) {
        result.pop();
        result.push(top.toUpperCase()); // identical letters fuse
      } else {
        result.push(letter);
      }
    }
    return result;
  }

  function foldColumns(direction) {
    const h = board.length;
    const w = board[0].length;
    const foldedWidth = Math.ceil(w / 2);
    const next = Array.from({ length: h }, () => Array.from({ length: foldedWidth }, () => []));

    for (let r = 0; r < h; r++) {
      for (let c = 0; c < w; c++) {
        const targetCol = direction === "left" ? Math.min(c, w - 1 - c) : (c >= Math.floor(w / 2) ? c - Math.floor(w / 2) : w - 1 - c - Math.floor(w / 2));
        const moving = direction === "left" ? c >= foldedWidth : c < Math.floor(w / 2);
        const pile = board[r][c];
        if (!moving) next[r][targetCol] = pile.slice();
        else next[r][targetCol] = drop(next[r][targetCol], pile);
      }
    }
    board = next;
  }

  function foldRows(direction) {
    const h = board.length;
    const w = board[0].length;
    const foldedHeight = Math.ceil(h / 2);
    const next = Array.from({ length: foldedHeight }, () => Array.from({ length: w }, () => []));

    for (let r = 0; r < h; r++) {
      for (let c = 0; c < w; c++) {
        const targetRow = direction === "up" ? Math.min(r, h - 1 - r) : (r >= Math.floor(h / 2) ? r - Math.floor(h / 2) : h - 1 - r - Math.floor(h / 2));
        const moving = direction === "up" ? r >= foldedHeight : r < Math.floor(h / 2);
        const pile = board[r][c];
        if (!moving) next[targetRow][c] = pile.slice();
        else next[targetRow][c] = drop(next[targetRow][c], pile);
      }
    }
    board = next;
  }

  for (const direction of folds) {
    if (direction === "left" || direction === "right") foldColumns(direction);
    else if (direction === "up" || direction === "down") foldRows(direction);
    else throw new RangeError(`unknown fold direction: ${direction}`);
  }

  // Return only the top-view letter (top of each pile), or underscore if empty.
  return board.map((row) => row.map((pile) => pile.length ? pile[pile.length - 1] : "_"));
}

if (typeof module !== "undefined" && module.exports) module.exports = squeezeMatrix;

