# Rock-Paper-Scissors
def player(prev_play, opponent_history=[], player_history=[]):
    if prev_play != '':
        opponent_history.append(prev_play)
    else:
        # Reset history for new match
        opponent_history.clear()
        player_history.clear()

    # Define helper functions
    def beat(move):
        return {'R': 'P', 'P': 'S', 'S': 'R'}.get(move, 'R')

    # Check if opponent is Kris (responds to player's last move)
    kris_pattern = len(opponent_history) >= 2 and len(player_history) >= 1
    if kris_pattern:
        for i in range(1, len(opponent_history)):
            if i >= len(player_history):
                break
            expected = beat(player_history[i-1])
            if opponent_history[i] != expected:
                kris_pattern = False
                break

    if kris_pattern and len(player_history) >= 1:
        # Counter Kris's strategy
        last_player_move = player_history[-1]
        kris_expected = beat(last_player_move)
        ideal_move = beat(kris_expected)
        player_history.append(ideal_move)
        return ideal_move

    # Build Markov model for opponent's moves
    n = 5
    model = {}
    for i in range(len(opponent_history)):
        for seq_len in range(1, n + 1):
            if i >= seq_len - 1:
                seq = tuple(opponent_history[i - seq_len + 1:i + 1])
                if i + 1 < len(opponent_history):
                    next_move = opponent_history[i + 1]
                    if seq not in model:
                        model[seq] = {}
                    model[seq][next_move] = model[seq].get(next_move, 0) + 1

    # Predict using the longest matching sequence
    predicted_move = None
    for seq_len in range(min(n, len(opponent_history)), 0, -1):
        seq = tuple(opponent_history[-seq_len:])
        if seq in model:
            next_moves = model[seq]
            predicted_move = max(next_moves, key=lambda k: next_moves[k])
            break

    # Fallback to most frequent move
    if not predicted_move:
        if opponent_history:
            freq = {}
            for move in opponent_history:
                freq[move] = freq.get(move, 0) + 1
            predicted_move = max(freq, key=lambda k: freq[k])
        else:
            predicted_move = 'R'

    # Choose move to beat predicted move
    ideal_move = beat(predicted_move)
    player_history.append(ideal_move)
    return ideal_move
