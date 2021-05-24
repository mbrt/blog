---
title: "Engineering a chess match against my brother"
date: 2021-05-24
tags: [chess, data-science]
toc: true
draft: true
---

This is the story of me trying to win a game of chess against my brother. A
single freaking game. What's so special about it? Am I good at chess? Not at
all. Did I improve at my game in the process? Also no. Is it a story about “the
journey rather than the destination”? Not really. Did I at least have fun in
the process? Not so sure. This is the story of me trying to be unconventional
at probably the most studied game in existence and using my software
engineering background for something that probably doesn't need it.

Although I'm a total disaster at chess and this post is useless to whoever
seriously wants to improve at their game, I still thought it was worth sharing
how it's possible to apply software engineering principles to a problem. Was I
successful? You'll see at the end.

## Intro: why I got into chess

During the 2020 COVID19 pandemic, my brother, along with many other people,
took a passion for playing online chess. After playing for a couple of months
he started speaking very enthusiastically about it and challenging other family
members, but where my father would budge (and get digitally butchered) I
wouldn't. For one thing, I refrained myself from delving into a potentially
very time consuming hobby. I knew enough about chess to understand that to be
only a mediocre amateur club player you'd still need to sink hundreds if not
thousands of hours in the game. I admit I also didn't like the thought of
losing against my brother, which was certain at the time, since he already had
hundreds of games under his belt and I had none.

Then one day I finally gave in and accepted a challenge from him. Needless to
say that I lost completely. I knew the rules and the rudimentary of the game,
having played a little bit as a kid, but my brother was obviously no match.
Looking at the post-game analysis in [chess.com](https://www.chess.com) I saw
that my disadvantage only grew, move by move, until reaching a +9 evaluation
(equivalent to having lost one rook, a bishop and a pawn to zero), which is
beyond any hope for a comeback, where I resigned. This blueprint was followed
for another couple of matches, where I understood I had to do something to
avoid being demolished every time.

This is when I decided I wanted to study the game a bit more.

## First attempt: learn

My first attempt in trying to improve at the game was to do the obvious: head
to Reddit and YouTube to see what other learners recommend. Between a tutorial
from [GM Naroditsky](https://www.youtube.com/channel/UCHP9CdeguNUI-_nBv_UXBhw),
some reading and puzzle solving on [Lichess](https://lichess.org) I also played
a few games with random people on the Internet. My rating stayed pretty low
nevertheless (1300 - 1400 Rapid on Lichess).

![Rating distribution](rating-distr.png)

After another couple of lost matches against my brother, it dawned on me that I
had no hope of beating him. I was following his same steps to improve (playing,
studying the game, watching videos), but I was dedicating much less time. At
that point he was playing hundreds of games a month and I was willing to play
maybe 10. At that pace I was only going to get further and further behind.

It was at that point that I had my second realization: I didn't really care
about the game, I didn't want to really improve at chess in general, I only
cared about beating a single person: my brother.

## Second attempt: study the opponent

A chess match can be generally divided into three phases: the
[opening](https://en.wikipedia.org/wiki/Chess_opening), the
[middlegame](https://en.wikipedia.org/wiki/Chess_middlegame) and the
[endgame](https://en.wikipedia.org/wiki/Chess_endgame). Converting a
significant advantage into a victory during an endgame is usually easy, after
studying some
[basic checkmate patterns](https://en.wikipedia.org/wiki/Checkmate#Basic_checkmates),
so the question for me was how to get that advantage in the first place.
Gaining advantage during a middlegame is usually achieved by long-term
[strategy](https://en.wikipedia.org/wiki/Chess_strategy) and
[tactics](https://en.wikipedia.org/wiki/Chess_tactic). The first can be
improved by reading and studying the game principles (something I can enjoy)
while the second is only possible by doing
[puzzles](https://en.wikipedia.org/wiki/Chess_puzzle) (which I don't
particularly like). I knew that I would be at a disadvantage here, given that
my brother used to do about 20 puzzles a day on chess.com, something I would
never be able to catch up to. This only left one possibility: gaining advantage
during the opening.

Chess opening theory is humongous and involves memorizing long sequences and
variations of moves, along with possible replies from the opponent. Beginners
don't really need to memorize much, but some familiarity with the most common
openings can go a long way (or so I was told). What I tried then was to look at
some random games that my brother played and try to study the openings he was
using. I looked at the [Italian opening](https://lichess.org/study/vJsZScnC)
and [Sicilian defense](https://lichess.org/study/jsSks17H) on Lichess and tried
to memorize the basic ideas behind those. I also watched a bunch of videos on
YouTube.

Obviously my brother did all of this already before me (better) and so I
understandably lost again. Not to mention that memorizing meaningless (at least
to me) opening moves is boring and laborious. I didn't really have fun doing
any of that. Another issue was that after my opponent deviated from the known
book moves I had no idea about how to react, because I didn't really understand
the positions.

It was time to step back and think again. I realized then I wasn't really
trying to beat my brother, but I was trying to improve my game against
opponents that played his same openings perfectly. Could I be more specific?
Could I prepare against my brother's weaknesses instead? Obviously this
wouldn't work against any player other than him, but that still satisfied my
goal.

## Third attempt: engineering

The problem to solve became: find positions out of the opening that my brother
(PlayerX from now on, for simplicity) would likely reach and be at a
disadvantage. Remember that neither of us is an expert of the game and at our
level players don't play very accurately. The only way to play against a good
player would be to follow book moves accurately, because you at least know in
advance there's no move they can make to get an advantage. The story is
different when you play against a club player. You can take risks (i.e. be
temporarily at a disadvantage) if you know that your opponent is unlikely to be
able to find the correct response and so get into trouble.

I also had a list of over 500 games my brother played on chess.com. Being a
software engineer it came natural to me approaching this as any other
engineering problem.

I started by downloading the games he played by using the
[chess.com APIs](https://www.chess.com/news/view/published-data-api) and split
them between black and white games. I focused on the games he played as black,
given that I felt I had better chances at guiding the game where I wanted if I
played as white.

```python
import json
import requests

def get_month_games(player, yyyy_mm):
    url = 'https://api.chess.com/pub/player/{}/games/{}'
    r = requests.get(url.format(player, yyyy_mm))
    if not r.ok:
        raise Exception('get_month_games failed')
    games = json.loads(r.content)
    # Format: {games: [{url, pgn}, ...]}
    return games['games']

# ...
```

```python
import chess.pgn
import io
import json

with open('games.json') as f:
    data = json.load(f)

games = []
for game in data:
    pgn = io.StringIO(game)
    games.append(chess.pgn.read_game(pgn))

black_games = [g for g in games if g.headers["Black"] == "playerx"]
```

Then I formulated the problem in this way: "Given all the positions PlayerX has
seen, what are the ones that he's likely to reach out of the opening where he
is at a substantial disadvantage?".

Now I had a well formulated problem and I was finally playing in a domain I was
familiar with. I decided to do my analysis in Python and in particular to use a
[Jupyter notebook](https://jupyter.org/), because I didn't really want to
create a reusable tool, but only to explore the data and find one solution for
the data I had available. It turns out Python already has pretty amazing
libraries to manipulate chess games:
[python-chess](https://python-chess.readthedocs.io/) (moves generation,
validation, visualization) and [python
stockfish](https://pypi.org/project/stockfish/) (bindings to evaluate a chess
position by using the very famous Stockfish engine).

I translated the problem into a graph problem in this way: A node is a
particular chess position (described in
[FEN notation](https://en.wikipedia.org/wiki/Forsyth%E2%80%93Edwards_Notation)).
An edge links two nodes where the destination is reachable from the source
position by doing a valid move. There's a special initial node that is common
to all games: the initial position.

I then constructed the graph of all games played by PlayerX as black,
additionally annotating every edge with the number of times the particular move
was played. What results is weighted directed graph (not a tree because a
position can be reached with different sequences of moves) similar to this one:

![Graph with counters](graph-freq-eval.svg)

Here the initial position is the squared node, the color indicates whether
from that position it's black or white's turn.

I also wanted then an evaluation of each position in terms of advantage for
white and to do so I used Stockfish. Given that the process of evaluating
thousands of positions is somewhat time consuming, I decided to do that
separately and create a JSON object mapping each unique FEN position to its
Stockfish evaluation.

[snippet stockfish eval all positions]

The evaluation is returned in centipawn advantage [TODO link] or mate in X
moves, where a positive number means advantage for white and negative is an
advantage for black:

```python
{"type":"cp", "value":12}    # 12 centipawns advantage for white.
{"type":"mate", "value":-3}  # Black has mate in three.
```

100 centipawns represent the advantage of having one more pawn than your
opponent and 300 is a minor piece like a bishop. Note however that Stockfish
assigns a value to pieces depending on their position, so it's entirely
possible to have an advantage of 1000 even if the pieces on the board are
equal.

I needed to map this evaluation into something more manageable, like a number
between 0 and 1. To do so, I decided arbitrarily that an advantage of 300+ is
mapped to 1.0 and a disadvantage of 300+ is mapped to 0. Additionally, any mate
in X (even if X is 20) is 1 or 0.

[snippet of computing rating from stockfish eval]

The information was then all there, I just needed to find nodes in the graph
(i.e. positions) where black was at a disadvantage, along with the sequence of
moves that was most likely to reach it. I needed to weigh the edges in such a
way that it was possible to easily compute the probability to reach a certain
position. My reasoning was as follow:

* At every position, we can evaluate the probability of doing a certain move by
  dividing the number of times the corresponding edge was taken by the total
  number of moves done from that position.
* Every edge will now have a weight between 0 and 1, where the higher the
  number, the higher the probability the edge will be taken from that position.
* The probability of a certain path is then the product of the probability of
  all the traversed edges.

To solve the problem with standard graph algorithms I needed to transform the
weights at the edges in such a way that:

* They represent a distance instead of a probability (i.e. the higher the
  distance, the lower the probability of the path).
* The distance between two nodes is the sum of the weights of the traversed
  edge (as opposed to the product of probabilities).

This is actually easier to do than to explain. The actual formula is very
simple:

```
distance(e) = -log(prob(e))
```

Taking the logarithm of the probability of an edge will give a negative number
by design, given that the probability is between 0 and 1. We don't have to
worry about the case of probability zero (which would shoot the logarithm to
minus infinity), as every edge of the graph has been taken at least once. The
lower the probability, the more negative the logarithm will be, so inverting
its sign will make it satisfy our requirements, because:

* The sum of logarithms is the same as the logarithm of the product of their
  arguments: `log(a) + log(b) = log(a*b)`.
* The bigger the result, the lower the underlying probability.

[graph of final weights]

Equipped with this data, we can compute the [shortest
path](https://en.wikipedia.org/wiki/Shortest_path_problem) between the initial
node and all other nodes by using
[Dijkstra's algorithm](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm).
The result is a mapping between every node and the shortest path to the initial
position, which represents the sequence of moves most likely to land in that
position.

At that point I arbitrarily chose a minimum advantage and sorted the paths by
probability. The first few paths represented my best chances to gain an
advantage out of the opening against PlayerX.





<!--
<iframe src="https://lichess.org/embed/2qKKl2MI?theme=auto&bg=auto" width=600 height=397 frameborder=0></iframe>
-->