---
title: Advent of Code 2020 (Part 1)
classes: wide
date: 2021-01-04 20:32
summary: A much-needed combo of learning and fun leading up to Christmas 2020
gallery:
  - image_path: /assets/images/AOC.png
    alt: "The Advent of Code website"
  - image_path: /assets/images/AOCDay1.png
    alt: "Opening text from Day 1"
---

After years of telling myself "_I really should start using Python_" but never quite committing to the change, I have been working almost exclusively in Python for the last year or so. Nevertheless, I have subsisted mainly on cannibalising snippets of code from elsewhere or else just making a mess of things as I gradually pick up the basics. So when my colleague [Mike Ratford](http://github.com/mratford){:target="_blank"} alerted the rest of the Data Science Hub (DaSH) to the existence [Advent of Code](https://adventofcode.com/2020/about){:target="_blank"}, it seemed like a perfect opportunity to tackle some simple problems from scratch. 

{% include gallery caption="The completed Advent of Code map (left), showing the epic '_journey_' our character is making over the course of 25 days and 25 puzzles, following the plot introduced on Day 1 (right)" %}

AoC is an annual series of programming puzzles, released daily from the 1st to the 25th December. The puzzles are each split into two parts, with the first setting up a scenario before the second part often ramps up the difficulty. Registered users are given their own bespoke input data to prevent copying their neighbour's homework (but that's not to say you can't run their code on your data) and upon submitting a correct answer you are rewarded with a gold star :star:. You can use any language you like, as long as you input the correct answer into the webpage at the end of it. 

For those for whom the challenge of simply completing all 25 puzzles isn't enough, a further incentive is given to the first 100 competitors to earn each star (often within minutes of the puzzles being published at midnight EST/UTC-5). 1st place receives 100 points, 2nd place 99, and so on. After this battle is concluded, a megathread is opened on the Advent of Code [subreddit](https://www.reddit.com/r/adventofcode/){:target="_blank"} where solutions can be posted and discussed.

{% include figure image_path="/assets/images/AOCleaderboard.png" alt="" caption="The MoJ Data Science Hub's private leaderboard on Advent of Code" %}

We mere mortals in the DaSH team set up a Slack channel to discuss our solutions, and a private leaderboard on the AoC website. Out of the 12 of us who signed up, the first to earn each star would receive 12 points, and so on down to 1 point for the last to solve a problem. Maybe without this element of competition, accountability and camaraderie I never would have persevered for the full month, and it was really enjoyable to learn about how my colleagues tackled a particular problem (mostly in Python but some in R). 

Unsurprisingly, as the problems became more taxing, not everyone was willing to commit to completing every puzzle, so it turned into a battle between myself and seasoned AoC-er Mike. Thankfully, I had built up an early lead simply by starting my day with each puzzle and finishing my code off over lunch if necessary, while others were late risers or simply had better things to do with their time!

My next post will go into the details of what new Python features/techniques I learned about in the process of solving these puzzles, not to mention the other methods Mike and others used that I'm sure to look into in future. For now though, here's my retrospective diary of how I tackled my very first Advent of Code epic journey:


## [Day 1 - Report Repair](https://adventofcode.com/2020/day/1){:target="_blank"}
[_My solution_](https://github.com/samnlindsay/advent_of_code/blob/main/Day01-ReportRepair.ipynb){:target="_blank"}

Kicking off the journey to our tropical island (if only!) we need to help out "the Elves in accounting" with an error in our "expense report" with a classic 2-sum followed by a 3-sum. Find the two entries in the `input` that sum to 2020 (Part 1) and then the _three_ entries that sum to 2020 (Part 2). I was happy to have a better idea than just brute forcing it (which can be tempting), and looked for a set intersection between the input list and `diff`:

```python
# Read numbers from text file (stripping newline chars and converting to int)
numbers = [int(e.strip()) for e in open("inputs/01-input.txt").readlines()]

# Corresponding numbers required to sum to 2020
diffs = [2020 - n for n in numbers]

# Numbers that occur in both lists 
# (i.e. that can be added to another number in the list to make 2020)
matches = list(set(numbers) & set(diffs))
```

Part 2 was then solved by wrapping this process into a function and applying it to find pairs of numbers (a,b) in `input` that sum to a value (c) in `diffs` such that a+b+c = 2020.

So far so simple...


## [Day 2 - Password Philosophy](https://adventofcode.com/2020/day/2){:target="_blank"}
[_My solution_](https://github.com/samnlindsay/advent_of_code/blob/main/Day02-PasswordPhilosophy.ipynb){:target="_blank"}

Now we must help validate passwords with given in the form "`1-3 a: abcde`" in the input data - a validation rule followed by the password itself after the colon. Part 1 interprets this example as "`a` must occur between `1` and `3` times in `abcde`", and in Part 2 we instead check that "the `1`st or `3`rd letter (but not both) of `abcde` must be `a`".

At first I converted the input into a pandas DataFrame using `str.split()` multiple times to create 4 columns (`min`,`max`,`letter`,`pw`), before `apply`ing a function `is_valid()` taking these 4 values as arguments - all a bit unnecessary and unsatisfactory. So I went back and used regular expressions instead, parsing a line of input and validating in one go:

```python
def is_valid_pt1(line:str): 
    # Identify 4 groups in regex (min, max, letter, password)
    l_min, l_max, l, pw = re.search(r'(\d+)-(\d+) (\w): (\w+)', line).groups()
    # Count repititons of letter
    rep = len(re.findall(l, pw))
    # Check if count between min and max
    valid = int(l_min) <= rep <= int(l_max)
    
    return valid

  def is_valid_pt2(line:str): 
    # Identify 4 groups in regex (index 1, index 2, letter, password)
    ind1, ind2, l, pw = re.search(r'(\d+)-(\d+) (\w): (\w+)', line).groups()
    # Check the characters in password at index 1 and 2
    m1 = pw[int(ind1)-1] == l
    m2 = pw[int(ind2)-1] == l
    # Check if only one matches
    valid = m1 != m2
    
    return valid
```

## [Day 3 - Toboggan Trajectory](https://adventofcode.com/2020/day/3){:target="_blank"}
[_My solution_](https://github.com/samnlindsay/advent_of_code/blob/main/Day03-TobogganTrajectory.ipynb){:target="_blank"}

Now things get a bit more imaginative. The input text is a wall of `.`s and `#`s representing a repeating grid of "trees" (`#`) in an area through which we must ride our toboggan to reach our destination. Starting at the top left, we must trace a path through the area, counting the trees we encounter en route through an area defined like the following:

```
..##.........##.........##.........##.........##.........##.......  --->
#...#...#..#...#...#..#...#...#..#...#...#..#...#...#..#...#...#..
.#....#..#..#....#..#..#....#..#..#....#..#..#....#..#..#....#..#.
..#.#...#.#..#.#...#.#..#.#...#.#..#.#...#.#..#.#...#.#..#.#...#.#
.#...##..#..#...##..#..#...##..#..#...##..#..#...##..#..#...##..#.
..#.##.......#.##.......#.##.......#.##.......#.##.......#.##.....  --->
.#.#.#....#.#.#.#....#.#.#.#....#.#.#.#....#.#.#.#....#.#.#.#....#
.#........#.#........#.#........#.#........#.#........#.#........#
#.##...#...#.##...#...#.##...#...#.##...#...#.##...#...#.##...#...
#...##....##...##....##...##....##...##....##...##....##...##....#
.#..#...#.#.#..#...#.#.#..#...#.#.#..#...#.#.#..#...#.#.#..#...#.#  --->
```

In Part 1 we move right 3 positions (`r=3`) for every 1 step down (`d=1`), but then in Part 2 we must try several different slopes. Reading the grid as a list of `strings` (each list element is a string representing a row of the grid), counting trees at integer positions with a given slope can be generalised easily enough:

```python
def count_trees(r,d):
    # Grid height/width
    h = len(strings)
    w = len(strings[0])

    slope = r/d
    
    count = sum([strings[y][(int(slope * y)) % w]=="#" for y in range(0,h,d)])

    return(count)   

slopes = [(1,1), (3,1), (5,1), (7,1), (1,2)]

trees = [count_trees(r,d) for r,d in slopes]
```

## [Day 4 - Passport Processing](https://adventofcode.com/2020/day/4){:target="_blank"}
[_My solution_](https://github.com/samnlindsay/advent_of_code/blob/main/Day04-PassportProcessing.ipynb){:target="_blank"}

Back to validating stuff, in the form of a "passport" consisting of field:value pairs, e.g.:
```
ecl:gry pid:860033327 eyr:2020 hcl:#fffffd
byr:1937 iyr:2017 cid:147 hgt:183cm
```

First we have to count passports with all the required fields, then we have to count those where all values are valid according to the set of rules provided (e.g. birth year `byr` being a 4-digit number between `1920` and `2002`).

Nothing exciting to see here (and my solution wasn't especially succinct or neat): 
  - input text -> list of `dict`s
  - check each dict for the required keys
  - check each value against the validation rule for that field 

## [Day 5 - Binary Boarding](https://adventofcode.com/2020/day/5){:target="_blank"}
[_My solution_](https://github.com/samnlindsay/advent_of_code/blob/main/Day05-BinaryBoarding.ipynb){:target="_blank"}

I enjoyed this one! I unexpectedly managed to answer Part 1 while having breakfast, without Python or my laptop.

Given a list of "boarding passes" with the seat location given by a string like `"FBFBBFFRLR"`. Each character divides the seats in two, selecting the `F`ront or `B`ack half (first 7 characters) successively to indentify a single row, and then selecting the `L`eft or `R`ight half of the seats on this row 3 times to select a single seat. Rows are numbered front to back starting from 0, and columns of seats numbered left to right from 0. The "seat ID" is then `row * 8 + column`.

Given a list of these boarding passes, the challenge is to identify the highest seat number on the plane (Part 1) and then find the one missing seat ID in the list (Part 2).

The reason I smugly answered Part 1 without any code is that I realised that the seat ID was just the decimal conversion of the boarding pass read as binary (F/L = 0, B/R = 1). No need for distinguishing between the row portion and the column portion of the string, or calculating seat ID from separate calculations of the row and column numbered (as several of my colleagues did, recognising each portion as binary, but not the seat ID itself).

So if `"FBFBBFFRLR"` corresponds to `0101100101` or `357`, the highest seat ID is simply the string with the most `B`s  at the start. Sorting the input strings alphabetically, `"BBFBFFFLLL"` tops the list, which can I can just about convert to decimal in my head: `1101000000` -> `512 + 256 + 64 = 832`. 

Part 2 I solved by sorting the list of seat IDs, and iterating over pairs of IDs looking for a pair with a gap of 2 (i.e. where there was a missing seat in the middle. This did the job in just a couple of lines of code, but I couldn't resist this minimal solution inspired by a couple of functions Mike used in his solution:

```python
import numpy as np 
seats = [int(e.strip().translate(str.maketrans("FBLR", "0101")),2)  for e in open("inputs/05-input.txt")]
print (f"Answer 1: {max(seats)}\nAnswer 2: {np.setdiff1d(range(min(seats),max(seats)+1),seats)[0]}")
```

I will mention `str.maketrans` in a subsequent post on the many Python tricks I learned in the process of doing AoC, but it is a really elegant way of performing that binary conversion. The `setdiff1d` function was also a really nice way of finding a seat `n` in the list where `n+1` was not. 

## [Day 6 - Custom Customs](https://adventofcode.com/2020/day/6){:target="_blank"}
[_My solution_](https://github.com/samnlindsay/advent_of_code/blob/main/Day06-CustomCustoms.ipynb){:target="_blank"}

Because the fun of travelling doesn't stop with passports and boarding passes, now we have to study some customs declaration forms. 26 yes-or-no questions (`a`-`z`) are asked of each passenger in a group. Their "yes" answers recorded as a string (e.g. `"abcx"`), with each passender on a new line, and each group separated by an blank line. For each group, we must find the number of questions answered "yes" by _any_ of the passengers (Part 1) and by _all_ passengers (Part 2), then calculating the sum over all groups.

Read the input file as a list of lists of strings (`answers`) to group the answer strings, each part of of the problem can be solved with one line:
```python
# Part 1 - Count unique characters in each group's answers
count = [len(set("".join(grp))) for grp in answers]
# Part 2 - Count characters common to ALL members of group
len(set.intersection(*[set(a) for a in grp])) for grp in answers 
```