<!DOCTYPE html>
<html lang="vi">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Chess Game with Advanced AI & Castling</title>
    <!-- Font: Lobster cho rank/file, Cinzel cho modal -->
    <link
      href="https://fonts.googleapis.com/css2?family=Lobster&family=Cinzel&display=swap"
      rel="stylesheet"
    />
    <!-- Canvas-confetti cho hiệu ứng pháo hoa -->
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.5.1/dist/confetti.browser.min.js"></script>
    <style>
      /* ===== CSS CHUNG ===== */
      body {
        margin: 0;
        padding: 0;
        background: url("https://i.pinimg.com/736x/dc/e3/cb/dce3cb7b2daeb86ca5bd921ae06f3b2f.jpg")
          no-repeat center center fixed;
        background-size: cover;
        font-family: Arial, sans-serif;
        color: #fff;
      }
      #board-container {
        margin: 20px auto;
        width: 500px;
        position: relative;
        z-index: 1;
      }
      #board {
        margin: 0 auto;
        display: inline-block;
        border: 2px solid #333;
      }
      table {
        border-collapse: collapse;
      }
      td {
        width: 60px;
        height: 60px;
        text-align: center;
        vertical-align: middle;
        font-size: 36px;
        cursor: pointer;
        position: relative;
      }
      td.rank,
      td.file {
        background: none;
        border: none;
        font-family: "Lobster", cursive;
        font-size: 24px;
        color: #fff;
        cursor: default;
      }
      .light-square {
        background-color: #f0d9b5;
      }
      .dark-square {
        background-color: #b58863;
      }
      .selected {
        outline: 3px solid yellow;
      }
      /* Marker hiển thị nước đi */
      .marker {
        position: absolute;
        width: 20px;
        height: 20px;
        border-radius: 50%;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        pointer-events: none;
      }
      .legal-marker {
        background-color: rgba(0, 255, 0, 0.6);
      }
      .capture-overlay {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        border: 3px solid rgba(255, 0, 0, 0.8);
        border-radius: 50%;
        box-sizing: border-box;
        pointer-events: none;
      }
      .in-check {
        box-shadow: 0 0 10px 3px red inset;
      }
      .white-piece {
        color: white;
        text-shadow: 1px 1px 2px black;
      }
      .black-piece {
        color: black;
        text-shadow: 1px 1px 2px white;
      }
      /* Modal chiến thắng */
      #modal {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 0, 0, 0.8);
        display: none;
        align-items: center;
        justify-content: center;
        z-index: 1100;
        flex-direction: column;
      }
      #modal-content {
        background-color: #fff;
        padding: 30px;
        border-radius: 10px;
        font-size: 24px;
        font-family: "Cinzel", serif;
        color: #333;
        text-shadow: 1px 1px 2px #aaa;
        animation: scaleIn 0.5s forwards;
        margin-bottom: 20px;
      }
      #play-again {
        padding: 10px 20px;
        font-size: 18px;
        font-family: "Cinzel", serif;
        cursor: pointer;
        border: none;
        border-radius: 5px;
        background-color: #333;
        color: #fff;
        box-shadow: 0 2px 5px rgba(0, 0, 0, 0.5);
      }
      @keyframes scaleIn {
        0% {
          transform: scale(0);
        }
        100% {
          transform: scale(1);
        }
      }
      /* Modal phong hậu */
      #promotion-modal {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 0, 0, 0.7);
        display: none;
        align-items: center;
        justify-content: center;
        z-index: 1200;
        flex-direction: column;
      }
      #promotion-board {
        background-color: #fff;
        padding: 20px;
        border-radius: 10px;
        font-family: "Cinzel", serif;
        color: #333;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.5);
      }
      .promotion-option {
        margin: 10px;
        padding: 10px 20px;
        font-size: 20px;
        cursor: pointer;
        border: 2px solid #333;
        border-radius: 5px;
        background-color: #eee;
        transition: background-color 0.3s;
      }
      .promotion-option:hover {
        background-color: #ddd;
      }
    </style>
  </head>
  <body>
    <div id="board-container">
      <h1>Chess Game with Advanced AI & Castling</h1>
      <div id="game-info">
        Current Player: <span id="current-player"></span>
      </div>
      <div id="board"></div>
      <div id="message"></div>
    </div>

    <!-- Modal chiến thắng -->
    <div id="modal">
      <div id="modal-content"></div>
      <button id="play-again" onclick="restartGame()">Chơi Lại</button>
    </div>

    <!-- Modal phong hậu -->
    <div id="promotion-modal">
      <div id="promotion-board">
        <p>Chọn quân để phong hậu:</p>
        <div style="display: flex; justify-content: space-around">
          <div class="promotion-option" onclick="selectPromotion('q')">
            ♕ Hậu
          </div>
          <div class="promotion-option" onclick="selectPromotion('r')">
            ♖ Xe
          </div>
          <div class="promotion-option" onclick="selectPromotion('b')">
            ♗ Tượng
          </div>
          <div class="promotion-option" onclick="selectPromotion('n')">
            ♘ Mã
          </div>
        </div>
      </div>
    </div>

    <script>
      /******************************************
       * HÀM KIỂM TRA VUA BỊ CHIẾU
       ******************************************/
      function isKingInCheckForBoard(board, color) {
        let kingPos = null;
        for (let i = 0; i < 8; i++) {
          for (let j = 0; j < 8; j++) {
            const piece = board[i][j];
            if (piece && piece instanceof King && piece.color === color) {
              kingPos = [i, j];
              break;
            }
          }
          if (kingPos) break;
        }
        if (!kingPos) return true;
        for (let i = 0; i < 8; i++) {
          for (let j = 0; j < 8; j++) {
            const piece = board[i][j];
            if (piece && piece.color !== color) {
              if (piece.validMove([i, j], kingPos, board)) return true;
            }
          }
        }
        return false;
      }

      /******************************************
       * ĐỊNH NGHĨA CÁC LỚP QUÂN CỜ
       ******************************************/
      class Piece {
        constructor(color) {
          this.color = color;
          this.symbol = "";
          this.hasMoved = false; // để hỗ trợ kiểm tra nhập thành
        }
        validMove(start, end, board) {
          return false;
        }
      }

      class Pawn extends Piece {
        constructor(color) {
          super(color);
          this.symbol = color === "white" ? "♙" : "♟";
        }
        validMove(start, end, board) {
          const direction = this.color === "white" ? -1 : 1;
          const [sRow, sCol] = start;
          const [eRow, eCol] = end;
          if (sCol === eCol) {
            if (eRow === sRow + direction && board[eRow][eCol] === null)
              return true;
            if (
              !this.hasMoved &&
              eRow === sRow + 2 * direction &&
              board[sRow + direction][sCol] === null &&
              board[eRow][eCol] === null
            )
              return true;
            return false;
          }
          if (Math.abs(eCol - sCol) === 1 && eRow === sRow + direction)
            return (
              board[eRow][eCol] !== null &&
              board[eRow][eCol].color !== this.color
            );
          return false;
        }
      }

      class Rook extends Piece {
        constructor(color) {
          super(color);
          this.symbol = color === "white" ? "♖" : "♜";
        }
        validMove(start, end, board) {
          const [sRow, sCol] = start;
          const [eRow, eCol] = end;
          if (sRow !== eRow && sCol !== eCol) return false;
          if (sRow === eRow) {
            const step = eCol > sCol ? 1 : -1;
            for (let c = sCol + step; c !== eCol; c += step)
              if (board[sRow][c] !== null) return false;
          } else {
            const step = eRow > sRow ? 1 : -1;
            for (let r = sRow + step; r !== eRow; r += step)
              if (board[r][sCol] !== null) return false;
          }
          const target = board[eRow][eCol];
          return target === null || target.color !== this.color;
        }
      }

      class Knight extends Piece {
        constructor(color) {
          super(color);
          this.symbol = color === "white" ? "♘" : "♞";
        }
        validMove(start, end, board) {
          const dx = Math.abs(end[1] - start[1]);
          const dy = Math.abs(end[0] - start[0]);
          if ((dx === 2 && dy === 1) || (dx === 1 && dy === 2)) {
            const target = board[end[0]][end[1]];
            return target === null || target.color !== this.color;
          }
          return false;
        }
      }

      class Bishop extends Piece {
        constructor(color) {
          super(color);
          this.symbol = color === "white" ? "♗" : "♝";
        }
        validMove(start, end, board) {
          const dx = Math.abs(end[1] - start[1]);
          const dy = Math.abs(end[0] - start[0]);
          if (dx !== dy) return false;
          const stepX = end[1] > start[1] ? 1 : -1;
          const stepY = end[0] > start[0] ? 1 : -1;
          for (let i = 1; i < dx; i++) {
            if (board[start[0] + i * stepY][start[1] + i * stepX] !== null)
              return false;
          }
          const target = board[end[0]][end[1]];
          return target === null || target.color !== this.color;
        }
      }

      class Queen extends Piece {
        constructor(color) {
          super(color);
          this.symbol = color === "white" ? "♕" : "♛";
        }
        validMove(start, end, board) {
          const rook = new Rook(this.color);
          const bishop = new Bishop(this.color);
          return (
            rook.validMove(start, end, board) ||
            bishop.validMove(start, end, board)
          );
        }
      }

      class King extends Piece {
        constructor(color) {
          super(color);
          this.symbol = color === "white" ? "♔" : "♚";
        }
        validMove(start, end, board) {
          const dx = Math.abs(end[1] - start[1]);
          const dy = Math.abs(end[0] - start[0]);

          // Kiểm tra nước đi tối đa 1 ô
          if (dx <= 1 && dy <= 1) {
            const target = board[end[0]][end[1]];
            // Nếu ô đích có quân cùng màu => false
            if (target && target.color === this.color) return false;
            // Có thể di chuyển (kể cả ăn quân đối phương)
            return true;
          }

          // Kiểm tra nhập thành
          if (!this.hasMoved && dy === 0 && dx === 2) {
            // short castling (king side) dx=2
            // or long castling (queen side) dx=2 but direction depends on end col
            const [sRow, sCol] = start;
            const [eRow, eCol] = end;
            if (sRow !== eRow) return false; // phải cùng hàng
            const row = sRow;
            // short castling
            if (eCol === sCol + 2) {
              // Rook ở sCol+3
              const rook = board[row][sCol + 3];
              if (!rook || !(rook instanceof Rook) || rook.hasMoved)
                return false;
              // kiểm tra ô giữa king và rook trống
              for (let c = sCol + 1; c < sCol + 3; c++) {
                if (board[row][c] !== null) return false;
              }
              // Tạm thời ok
              return true;
            }
            // long castling
            if (eCol === sCol - 2) {
              // Rook ở sCol-4
              const rook = board[row][sCol - 4];
              if (!rook || !(rook instanceof Rook) || rook.hasMoved)
                return false;
              // kiểm tra ô giữa king và rook trống
              for (let c = sCol - 1; c > sCol - 4; c--) {
                if (board[row][c] !== null) return false;
              }
              return true;
            }
          }
          return false;
        }
      }

      /******************************************
       * LỚP CHESSGAME
       ******************************************/
      class ChessGame {
        constructor() {
          this.board = this.createEmptyBoard();
          this.currentPlayer = "white";
          this.gameOver = false;
          this.setupBoard();
        }
        createEmptyBoard() {
          const board = [];
          for (let i = 0; i < 8; i++) {
            board.push(new Array(8).fill(null));
          }
          return board;
        }
        setupBoard() {
          // Trắng
          this.board[7] = [
            new Rook("white"),
            new Knight("white"),
            new Bishop("white"),
            new Queen("white"),
            new King("white"),
            new Bishop("white"),
            new Knight("white"),
            new Rook("white"),
          ];
          for (let i = 0; i < 8; i++) {
            this.board[6][i] = new Pawn("white");
          }
          // Đen
          this.board[0] = [
            new Rook("black"),
            new Knight("black"),
            new Bishop("black"),
            new Queen("black"),
            new King("black"),
            new Bishop("black"),
            new Knight("black"),
            new Rook("black"),
          ];
          for (let i = 0; i < 8; i++) {
            this.board[1][i] = new Pawn("black");
          }
        }
        toNotation(row, col) {
          const file = String.fromCharCode("a".charCodeAt(0) + col);
          const rank = 8 - row;
          return file + rank;
        }
        parsePosition(pos) {
          if (pos.length !== 2) return null;
          const file = pos[0].toLowerCase();
          const rank = parseInt(pos[1]);
          if (file < "a" || file > "h" || isNaN(rank) || rank < 1 || rank > 8)
            return null;
          const col = file.charCodeAt(0) - "a".charCodeAt(0);
          const row = 8 - rank;
          return [row, col];
        }

        // Hàm thực hiện nhập thành
        handleCastling(sRow, sCol, eRow, eCol) {
          // short castling
          if (eCol === sCol + 2) {
            // di chuyển rook
            this.board[eRow][eCol - 1] = this.board[eRow][eCol + 1];
            this.board[eRow][eCol + 1] = null;
            this.board[eRow][eCol - 1].hasMoved = true;
          }
          // long castling
          else if (eCol === sCol - 2) {
            this.board[eRow][eCol + 1] = this.board[eRow][eCol - 2];
            this.board[eRow][eCol - 2] = null;
            this.board[eRow][eCol + 1].hasMoved = true;
          }
        }

        movePiece(startPos, endPos) {
          if (this.gameOver) return false;
          const start = this.parsePosition(startPos);
          const end = this.parsePosition(endPos);
          if (!start || !end) return false;
          const [sRow, sCol] = start;
          const [eRow, eCol] = end;
          const piece = this.board[sRow][sCol];
          if (!piece || piece.color !== this.currentPlayer) return false;
          if (!piece.validMove([sRow, sCol], [eRow, eCol], this.board))
            return false;

          // Clone board để kiểm tra
          const tempBoard = cloneBoard(this.board);
          tempBoard[eRow][eCol] = tempBoard[sRow][sCol];
          tempBoard[sRow][sCol] = null;
          // Nếu nước đi làm Vua bị chiếu => invalid
          if (isKingInCheckForBoard(tempBoard, this.currentPlayer)) {
            return false;
          }

          // Nếu là nhập thành
          if (
            piece instanceof King &&
            !piece.hasMoved &&
            Math.abs(eCol - sCol) === 2 &&
            sRow === eRow
          ) {
            this.handleCastling(sRow, sCol, eRow, eCol);
          }

          // Di chuyển chính thức
          this.board[eRow][eCol] = piece;
          this.board[sRow][sCol] = null;
          piece.hasMoved = true;

          // Nếu là Tốt đến hàng cuối => phong hậu
          if (piece instanceof Pawn) {
            if (
              (piece.color === "white" && eRow === 0) ||
              (piece.color === "black" && eRow === 7)
            ) {
              pendingPromotion = { row: eRow, col: eCol, color: piece.color };
              showPromotionModal();
              return true;
            }
          }

          // Kiểm tra mất Vua => gameOver
          if (!this.findKing("white") || !this.findKing("black")) {
            this.gameOver = true;
          }

          // Đổi lượt
          this.currentPlayer =
            this.currentPlayer === "white" ? "black" : "white";
          return true;
        }

        promotePawn(row, col, newPiece) {
          this.board[row][col] = newPiece;
        }
        findKing(color) {
          for (let i = 0; i < 8; i++) {
            for (let j = 0; j < 8; j++) {
              const piece = this.board[i][j];
              if (piece instanceof King && piece.color === color) return [i, j];
            }
          }
          return null;
        }
        isInCheck(color) {
          return isKingInCheckForBoard(this.board, color);
        }
      }

      /******************************************
       * HÀM HỖ TRỢ
       ******************************************/
      function cloneBoard(board) {
        const newBoard = [];
        for (let i = 0; i < 8; i++) {
          newBoard[i] = [];
          for (let j = 0; j < 8; j++) {
            const piece = board[i][j];
            if (!piece) newBoard[i][j] = null;
            else {
              let newPiece;
              if (piece instanceof Pawn) {
                newPiece = new Pawn(piece.color);
                newPiece.hasMoved = piece.hasMoved;
              } else if (piece instanceof Rook) {
                newPiece = new Rook(piece.color);
                newPiece.hasMoved = piece.hasMoved;
              } else if (piece instanceof Knight) {
                newPiece = new Knight(piece.color);
                newPiece.hasMoved = piece.hasMoved;
              } else if (piece instanceof Bishop) {
                newPiece = new Bishop(piece.color);
                newPiece.hasMoved = piece.hasMoved;
              } else if (piece instanceof Queen) {
                newPiece = new Queen(piece.color);
                newPiece.hasMoved = piece.hasMoved;
              } else if (piece instanceof King) {
                newPiece = new King(piece.color);
                newPiece.hasMoved = piece.hasMoved;
              }
              newBoard[i][j] = newPiece;
            }
          }
        }
        return newBoard;
      }

      /******************************************
       * AI: MINIMAX VỚI ALPHA-BETA PRUNING
       ******************************************/
      const AI_MAX_DEPTH = 3; // Độ sâu cố định => vừa nhanh vừa đủ thông minh

      function generateMoves(board, color) {
        const moves = [];
        for (let i = 0; i < 8; i++) {
          for (let j = 0; j < 8; j++) {
            const piece = board[i][j];
            if (piece && piece.color === color) {
              for (let m = 0; m < 8; m++) {
                for (let n = 0; n < 8; n++) {
                  if (piece.validMove([i, j], [m, n], board)) {
                    const newBoard = cloneBoard(board);
                    newBoard[m][n] = newBoard[i][j];
                    newBoard[i][j] = null;
                    if (!isKingInCheckForBoard(newBoard, color)) {
                      moves.push({ start: [i, j], end: [m, n] });
                    }
                  }
                }
              }
            }
          }
        }
        return moves;
      }

      function evaluateBoard(board) {
        let score = 0;
        const values = {
          Pawn: 1,
          Knight: 3,
          Bishop: 3,
          Rook: 5,
          Queen: 9,
          King: 10000,
        };
        const center = [
          [3, 3],
          [3, 4],
          [4, 3],
          [4, 4],
        ];
        // Tính điểm dựa trên giá trị quân + bonus
        for (let i = 0; i < 8; i++) {
          for (let j = 0; j < 8; j++) {
            const piece = board[i][j];
            if (piece) {
              const type = piece.constructor.name;
              let val = values[type] || 0;
              // Bonus vị trí trung tâm
              if (center.some((pos) => pos[0] === i && pos[1] === j))
                val += 0.5;
              // Giảm điểm nếu King ở biên
              if (
                piece instanceof King &&
                (i === 0 || i === 7 || j === 0 || j === 7)
              )
                val -= 2;
              // Tính vào score
              score += piece.color === "black" ? val : -val;
            }
          }
        }
        return score;
      }

      function minimax(board, depth, alpha, beta, maximizingPlayer) {
        const whiteKing = board.some((row) =>
          row.some((p) => p instanceof King && p.color === "white")
        );
        const blackKing = board.some((row) =>
          row.some((p) => p instanceof King && p.color === "black")
        );
        // Nếu hết quân King hoặc hết độ sâu => đánh giá
        if (depth === 0 || !whiteKing || !blackKing) {
          return evaluateBoard(board);
        }

        if (maximizingPlayer) {
          let maxEval = -Infinity;
          const moves = generateMoves(board, "black");
          // Move ordering đơn giản: sắp xếp nước đi theo evaluateBoard
          moves.sort((a, b) => {
            const newBoardA = cloneBoard(board);
            newBoardA[a.end[0]][a.end[1]] = newBoardA[a.start[0]][a.start[1]];
            newBoardA[a.start[0]][a.start[1]] = null;
            const newBoardB = cloneBoard(board);
            newBoardB[b.end[0]][b.end[1]] = newBoardB[b.start[0]][b.start[1]];
            newBoardB[b.start[0]][b.start[0]] = null;
            return evaluateBoard(newBoardB) - evaluateBoard(newBoardA);
          });
          for (const move of moves) {
            const newBoard = cloneBoard(board);
            newBoard[move.end[0]][move.end[1]] =
              newBoard[move.start[0]][move.start[1]];
            newBoard[move.start[0]][move.start[1]] = null;
            const evalScore = minimax(newBoard, depth - 1, alpha, beta, false);
            maxEval = Math.max(maxEval, evalScore);
            alpha = Math.max(alpha, evalScore);
            if (beta <= alpha) break;
          }
          return maxEval;
        } else {
          let minEval = Infinity;
          const moves = generateMoves(board, "white");
          moves.sort((a, b) => {
            const newBoardA = cloneBoard(board);
            newBoardA[a.end[0]][a.end[1]] = newBoardA[a.start[0]][a.start[1]];
            newBoardA[a.start[0]][a.start[1]] = null;
            const newBoardB = cloneBoard(board);
            newBoardB[b.end[0]][b.end[1]] = newBoardB[b.start[0]][b.start[1]];
            newBoardB[b.start[0]][b.start[1]] = null;
            return evaluateBoard(newBoardB) - evaluateBoard(newBoardA);
          });
          for (const move of moves) {
            const newBoard = cloneBoard(board);
            newBoard[move.end[0]][move.end[1]] =
              newBoard[move.start[0]][move.start[1]];
            newBoard[move.start[0]][move.start[1]] = null;
            const evalScore = minimax(newBoard, depth - 1, alpha, beta, true);
            minEval = Math.min(minEval, evalScore);
            beta = Math.min(beta, evalScore);
            if (beta <= alpha) break;
          }
          return minEval;
        }
      }

      /******************************************
       * AI CHO QUÂN ĐEN
       ******************************************/
      function aiMove() {
        if (game.currentPlayer !== "black" || game.gameOver) return;
        const moves = generateMoves(game.board, "black");
        if (moves.length === 0) return;
        let bestEval = -Infinity;
        let bestMove = moves[0];
        // Move ordering
        moves.sort((a, b) => {
          const boardA = cloneBoard(game.board);
          boardA[a.end[0]][a.end[1]] = boardA[a.start[0]][a.start[1]];
          boardA[a.start[0]][a.start[1]] = null;
          const boardB = cloneBoard(game.board);
          boardB[b.end[0]][b.end[1]] = boardB[b.start[0]][b.start[1]];
          boardB[b.start[0]][b.start[1]] = null;
          return evaluateBoard(boardB) - evaluateBoard(boardA);
        });

        for (const move of moves) {
          const newBoard = cloneBoard(game.board);
          newBoard[move.end[0]][move.end[1]] =
            newBoard[move.start[0]][move.start[1]];
          newBoard[move.start[0]][move.start[1]] = null;
          const evalScore = minimax(
            newBoard,
            AI_MAX_DEPTH,
            -Infinity,
            Infinity,
            false
          );
          if (evalScore > bestEval) {
            bestEval = evalScore;
            bestMove = move;
          }
        }

        const startPos = game.toNotation(bestMove.start[0], bestMove.start[1]);
        const endPos = game.toNotation(bestMove.end[0], bestMove.end[1]);
        game.movePiece(startPos, endPos);
        renderBoard();
        if (!game.findKing("white") || !game.findKing("black")) {
          game.gameOver = true;
          if (!game.findKing("white"))
            showModal("Chúc mừng đội đen đã chiến thắng!");
          else showModal("Chúc mừng đội trắng đã chiến thắng!");
        }
      }

      /******************************************
       * GIAO DIỆN & XỬ LÝ SỰ KIỆN
       ******************************************/
      let game = new ChessGame();
      let selectedSquare = null;
      let pendingPromotion = null;
      let fireworksInterval = null;

      function getLegalMovesForSelected(selected) {
        const legalMoves = [];
        if (!selected) return legalMoves;
        const [sRow, sCol] = selected;
        const piece = game.board[sRow][sCol];
        if (!piece) return legalMoves;
        for (let i = 0; i < 8; i++) {
          for (let j = 0; j < 8; j++) {
            if (piece.validMove([sRow, sCol], [i, j], game.board)) {
              // Kiểm tra an toàn cho Vua
              const tempBoard = cloneBoard(game.board);
              tempBoard[i][j] = tempBoard[sRow][sCol];
              tempBoard[sRow][sCol] = null;
              if (!isKingInCheckForBoard(tempBoard, piece.color)) {
                if (game.board[i][j] === null)
                  legalMoves.push({ row: i, col: j, type: "move" });
                else if (game.board[i][j].color !== piece.color)
                  legalMoves.push({ row: i, col: j, type: "capture" });
              }
            }
          }
        }
        return legalMoves;
      }

      function renderBoard() {
        const boardDiv = document.getElementById("board");
        let html = "<table>";
        const legalMoves = selectedSquare
          ? getLegalMovesForSelected(selectedSquare)
          : [];
        const whiteInCheck = game.findKing("white")
          ? game.isInCheck("white")
          : false;
        const blackInCheck = game.findKing("black")
          ? game.isInCheck("black")
          : false;
        for (let i = 0; i < 8; i++) {
          const rank = 8 - i;
          html += "<tr>";
          html += `<td class="rank">${rank}</td>`;
          for (let j = 0; j < 8; j++) {
            const piece = game.board[i][j];
            const cellClasses = [];
            cellClasses.push(
              (i + j) % 2 === 0 ? "light-square" : "dark-square"
            );
            if (
              selectedSquare &&
              selectedSquare[0] === i &&
              selectedSquare[1] === j
            )
              cellClasses.push("selected");

            // Đánh dấu ô Vua đang bị chiếu
            if (piece instanceof King) {
              const kingPos = game.findKing(piece.color);
              if (kingPos && kingPos[0] === i && kingPos[1] === j) {
                if (
                  (piece.color === "white" && whiteInCheck) ||
                  (piece.color === "black" && blackInCheck)
                )
                  cellClasses.push("in-check");
              }
            }

            let markerHTML = "";
            const lm = legalMoves.find((m) => m.row === i && m.col === j);
            if (lm) {
              if (lm.type === "move")
                markerHTML = '<div class="marker legal-marker"></div>';
              else if (lm.type === "capture")
                markerHTML = '<div class="capture-overlay"></div>';
            }

            let displayPiece = "";
            if (piece) {
              displayPiece =
                piece.color === "white"
                  ? `<span class="white-piece">${piece.symbol}</span>`
                  : `<span class="black-piece">${piece.symbol}</span>`;
            }
            html += `<td class="${cellClasses.join(
              " "
            )}" onclick="handleCellClick(${i}, ${j})">
                    ${markerHTML}
                    ${displayPiece}
                   </td>`;
          }
          html += "</tr>";
        }
        html += "<tr><td></td>";
        for (let j = 0; j < 8; j++) {
          const file = String.fromCharCode("a".charCodeAt(0) + j);
          html += `<td class="file">${file}</td>`;
        }
        html += "</tr></table>";
        boardDiv.innerHTML = html;
        document.getElementById("current-player").innerText =
          game.currentPlayer;

        // Kiểm tra checkmate
        if (checkCheckmate()) return;

        // Nếu đến lượt AI (đen) => di chuyển
        if (game.currentPlayer === "black" && !game.gameOver) {
          setTimeout(aiMove, 500);
        }
      }

      function checkCheckmate() {
        const movesCurrent = generateMoves(game.board, game.currentPlayer);
        if (movesCurrent.length === 0 && game.isInCheck(game.currentPlayer)) {
          game.gameOver = true;
          if (game.currentPlayer === "white")
            showModal("Đội đen đã chiếu bí! Đội đen thắng!");
          else showModal("Đội trắng đã chiếu bí! Đội trắng thắng!");
          return true;
        }
        return false;
      }

      function showModal(message) {
        const modal = document.getElementById("modal");
        const modalContent = document.getElementById("modal-content");
        modalContent.innerText = message;
        modal.style.display = "flex";
        startFireworks();
      }
      function hideModal() {
        const modal = document.getElementById("modal");
        modal.style.display = "none";
        stopFireworks();
      }
      function startFireworks() {
        fireworksInterval = setInterval(() => {
          confetti({
            particleCount: 100,
            spread: 100,
            origin: { y: 0.6 },
          });
        }, 500);
      }
      function stopFireworks() {
        if (fireworksInterval) {
          clearInterval(fireworksInterval);
          fireworksInterval = null;
        }
      }

      function showPromotionModal() {
        const promotionModal = document.getElementById("promotion-modal");
        promotionModal.style.display = "flex";
      }
      function hidePromotionModal() {
        const promotionModal = document.getElementById("promotion-modal");
        promotionModal.style.display = "none";
      }
      function selectPromotion(choice) {
        let newPiece;
        switch (choice) {
          case "r":
            newPiece = new Rook(pendingPromotion.color);
            break;
          case "b":
            newPiece = new Bishop(pendingPromotion.color);
            break;
          case "n":
            newPiece = new Knight(pendingPromotion.color);
            break;
          default:
            newPiece = new Queen(pendingPromotion.color);
            break;
        }
        game.promotePawn(pendingPromotion.row, pendingPromotion.col, newPiece);
        pendingPromotion = null;
        hidePromotionModal();
        if (!game.findKing("white") || !game.findKing("black")) {
          game.gameOver = true;
          if (!game.findKing("white"))
            showModal("Chúc mừng đội đen đã chiến thắng!");
          else showModal("Chúc mừng đội trắng đã chiến thắng!");
        } else {
          game.currentPlayer =
            game.currentPlayer === "white" ? "black" : "white";
        }
        renderBoard();
      }

      function handleCellClick(row, col) {
        if (game.currentPlayer === "black" || game.gameOver) return;
        const clickedPiece = game.board[row][col];
        if (!selectedSquare) {
          if (clickedPiece && clickedPiece.color === "white")
            selectedSquare = [row, col];
        } else {
          const [sRow, sCol] = selectedSquare;
          const selectedPiece = game.board[sRow][sCol];
          if (clickedPiece && clickedPiece.color === selectedPiece.color)
            selectedSquare = [row, col];
          else {
            const startPos = game.toNotation(sRow, sCol);
            const endPos = game.toNotation(row, col);
            const result = game.movePiece(startPos, endPos);
            document.getElementById("message").innerText = result
              ? "Move successful!"
              : "Invalid move!";
            selectedSquare = null;
            if (!game.findKing("white"))
              showModal("Chúc mừng đội đen đã chiến thắng!");
            else if (!game.findKing("black"))
              showModal("Chúc mừng đội trắng đã chiến thắng!");
          }
        }
        renderBoard();
      }

      function restartGame() {
        selectedSquare = null;
        game = new ChessGame();
        hideModal();
        document.getElementById("message").innerText = "";
        renderBoard();
      }

      // Khởi tạo giao diện
      renderBoard();
    </script>
  </body>
</html>
