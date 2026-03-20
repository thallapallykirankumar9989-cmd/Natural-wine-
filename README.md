<!DOCTYPE html>
<html lang="te">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>3D Chess Clash</title>
    <link rel="stylesheet" href="https://unpkg.com/@chrisoakman/chessboardjs@1.0.0/dist/chessboard-1.0.0.min.css">
    <style>
        body {
            margin: 0; padding: 0;
            background-color: #1a1a1a; /* డార్క్ థీమ్ */
            font-family: sans-serif;
            display: flex; justify-content: center;
            overflow: hidden; /* పేజీ కదలకుండా ఉండటానికి */
        }
        .app-container {
            width: 100%; max-width: 500px;
            height: 100vh; background: #2c3e50;
            display: flex; flex-direction: column;
        }
        .header {
            background: #111; color: #f1c40f;
            padding: 15px; text-align: center;
            font-size: 20px; font-weight: bold;
            border-bottom: 3px solid #f1c40f;
        }
        #board-wrapper {
            flex-grow: 1;
            display: flex; align-items: center; justify-content: center;
            padding: 10px;
        }
        /* 3D ఎఫెక్ట్ కోసం బోర్డ్ డిజైన్ */
        #myBoard {
            width: 95vw; max-width: 450px;
            box-shadow: 0 15px 35px rgba(0,0,0,0.5);
            border: 8px solid #34495e;
            border-radius: 5px;
        }
        .controls {
            padding: 20px; background: #111;
            display: flex; gap: 10px;
        }
        .status {
            color: white; text-align: center;
            padding: 10px; font-weight: bold; background: #222;
        }
        .btn {
            flex: 1; padding: 12px; border: none;
            border-radius: 5px; cursor: pointer;
            font-weight: bold; font-size: 16px;
        }
        .btn-undo { background: #e67e22; color: white; }
        .btn-reset { background: #e74c3c; color: white; }
    </style>
</head>
<body>

<div class="app-container">
    <div class="header">3D CHESS CLASH ♟️</div>
    
    <div class="status" id="status">White's Turn</div>

    <div id="board-wrapper">
        <div id="myBoard"></div>
    </div>

    <div class="controls">
        <button class="btn btn-undo" id="undoBtn">UNDO</button>
        <button class="btn btn-reset" id="resetBtn">RESET</button>
    </div>
</div>

<script src="https://code.jquery.com/jquery-3.5.1.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/chess.js/0.10.3/chess.min.js"></script>
<script src="https://unpkg.com/@chrisoakman/chessboardjs@1.0.0/dist/chessboard-1.0.0.min.js"></script>

<script>
    var board = null;
    var game = new Chess();
    var statusEl = $('#status');

    function onDragStart (source, piece) {
        if (game.game_over()) return false;
        if ((game.turn() === 'w' && piece.search(/^b/) !== -1) ||
            (game.turn() === 'b' && piece.search(/^w/) !== -1)) {
            return false;
        }
    }

    function onDrop (source, target) {
        var move = game.move({
            from: source,
            to: target,
            promotion: 'q'
        });
        if (move === null) return 'snapback';
        updateStatus();
    }

    function onSnapEnd () {
        board.position(game.fen());
    }

    function updateStatus () {
        var status = '';
        var moveColor = (game.turn() === 'w') ? 'White' : 'Black';

        if (game.in_checkmate()) {
            status = 'Game Over! ' + moveColor + ' is Checkmated.';
        } else if (game.in_draw()) {
            status = 'Game Over! Draw.';
        } else {
            status = moveColor + "'s Turn";
            if (game.in_check()) { status += ' (CHECK!)'; }
        }
        statusEl.html(status);
    }

    var config = {
        draggable: true,
        position: 'start',
        onDragStart: onDragStart,
        onDrop: onDrop,
        onSnapEnd: onSnapEnd,
        // హై-క్వాలిటీ 3D లాంటి పీసెస్
        pieceTheme: 'https://chessboardjs.com/img/chesspieces/wikipedia/{piece}.png'
    };

    board = Chessboard('myBoard', config);
    updateStatus();

    $('#undoBtn').on('click', function() {
        game.undo();
        board.position(game.fen());
        updateStatus();
    });

    $('#resetBtn').on('click', function() {
        game.reset();
        board.start();
        updateStatus();
    });

    // విండో రీసైజ్ అయినా బోర్డ్ కదలదు, సెట్ అవుతుంది
    $(window).resize(board.resize);
</script>

</body>
</html>
