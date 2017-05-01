def bid(hand, player_no, phase_no, deck_top, reshuffled=False, 
        player_data=None, suppress_player_data=True):
    """
    This function takes a hand (a list or tuple of cards as strings, 
    player_no, an int
    phase_no which is an integer and deck_top symbolising the card that
    dictates trump suit. There are some optional arguments, reshuffled
    which is by default False, showing whether the deck was shuffled.
    player_data is by default None, but can be information to be
    passed over the functions. (It should be the deck remaining)
    Suppress_player_data is by default True but
    is False if you are wanting to pass player_data. This returns an int
    of the bid the program wants to make. Yep
    """
    
    #creating a full deck 
    full_deck = []
    for suit in "SDHC":
        for value in "A234567890JQK":
            full_deck.append(value+suit)
            
            
    if reshuffled:
        deck = full_deck
        
    # my player_data is the remaining deck    
    if player_data is not None:
        deck = player_data
    else:
        deck = full_deck
        
        
    trump = deck_top[1]
    
    #these lines obey the game's rules about bids.
    if phase_no == 1 or phase_no == 19:
        for card in hand:
            if card[0] in "67890JQKA" or card[1] == trump:
                return 0         
        return 1
    elif phase_no == 10:
        return 0
    elif phase_no in [4, 16]:
        return 1
    elif phase_no in [8, 12]:
        return 2 

    #take out the cards which are available to get a more accurate deck
    for card in hand:
        if card in deck:
            deck.remove(card)
        
    # hacks will return a bid because there is a check_bid argument set
    # to one, it's the third last argument.
    return hacks(hand, None, (), deck, full_deck, player_no, 
                 trump, (), deck_top, 1, 11, player_data, suppress_player_data)


def is_valid_play(play, curr_trick, hand):
    """
    This function takes three arguments, play which is a string card you want
    to consider, curr_trick, a tuple with string elements of the current
    trick and lastly, hand which is a tuple of strings indicating your hand.
    The function returns True if you can make this play, and False otherwise.
    """
    
    #first check if play is in the hand
    if play in hand:
        
        # if there is no lead you can make any play
        try:
            lead = curr_trick[0][1]
        except IndexError:
            return True
        
        # checks if you have the lead suit, if you do then check whether
        # the play card is of the same suit.
        for card in hand:
            if card[1] == lead:
                if play[1] == lead:
                    return True
                return False
        return True
    
    else:
        return False


def score_phase(bids, tricks, deck_top, 
                player_data=None, suppress_player_data=True):
    """
    This function takes bids which is a tuple of integers representing
    the players' bids. It takes tricks, which is a tuple of tuples. It takes
    a string deck_top which dictates trump suit. player_data is the deck of
    cards and suppress_player_data controls whether player_data is accepted.
    The function returns a tuple of the scores of the players as integers.
    """
    
    trump = deck_top[1]
    
    #starter is the player leading
    starter = 0
    
    scores = [0 for i in range(len(bids))]
    
    for trick in tricks:
        lead = trick[0][1]
        
        # if trump and lead are the same, I assert there is no lead essentially
        if lead == trump:
            lead = None
            
        # the winner is the winning card    
        winner = best_card(trick, trump, lead)
        
        # the next starter is chosen here
        starter = (starter + trick.index(winner))%len(bids)
        scores[starter] += 1
        
    # this checks whether the players made their bid and adds 10 if so.   
    for count in range(len(bids)):
        if scores[count] == bids[count]:
            scores[count] += 10
    return tuple(scores)
    
def best_card(hand, trump, lead):
    """
    This is a helper function for my program. It takes hand a tuple of strings
    , trump which is the trump suit as a string and lead as a string.
    It returns the strongest card in this specific situation as a string.
    """
    
    values = {"2": 2, "3": 3, "4": 4, "5": 5, "6": 6, "7": 7, "8": 8, 
              "9": 9, "0": 10, "J": 11, "Q": 12, "K": 13, "A": 14}
    suit_list = ["S", "D", "H", "C"]
    suits = {}
    
    # this if and else area creates the order of suits, how powerful they are
    # compared to each other. IT is a dictionary. eg S:0 D:0 C:1 H:2, hearts
    # therefore is the trump and clubs is the lead.
    if lead is not None:
        for suit in suit_list:
            suits[suit] = 0
        suits[lead] = 1
        suits[trump] = 2
    else:
        for suit in suit_list:
            suits[suit] = 0
        suits[trump] = 2
    
    # The hand is casted to a list to make it mutable, then sorted by
    # suit and then sub sorted by value.
    hand = list(hand)
    hand = sorted(hand, key=lambda x: (suits[x[1]], 
                  values[x[0]]), reverse=True)
    return hand[0]


def play(curr_trick, hand, prev_tricks, player_no, deck_top, phase_bids, 
         player_data=None, suppress_player_data=True, 
         is_valid=is_valid_play, score=score_phase):
    """
    This function takes curr_trick, a tuple of strings of the curr trick
    hand - a tuple of strings
    prev_tricks - a tuple of tuples and each element inside the tuples are
    strings of cards
    player_no - an integer
    deck_top - a string of the card used to determine trump
    phase_bids -  tuple of integers
    player_data - set to None but can be used to pass extra data
    suppress_player_data - set to True but can be used to pass extra data
    is_valid_play and score_phase functions are imported.
    This returns a string of the card it wants to play
    """
    # creating a full deck
    full_deck = []
    for suit in "SDHC":
        for value in "A234567890JQK":
            full_deck.append(value+suit)
            
    # if there is player_data we'll use that as the deck        
    if player_data is not None:
        deck = player_data
    else:
        deck = full_deck
        
    trump = deck_top[1]
    
    # try create a lead, if there is no lead or if it the same as the trump
    # make the lead None
    try:
        lead = curr_trick[0][1]
        if lead == trump:
            lead = None
    except IndexError:
        lead = None
    

    # removing cards from the deck to show the remaining cards
    for card in hand:
        if card in deck:
            deck.remove(card)
    if deck_top in deck:
        deck.remove(deck_top)
    for cards in prev_tricks:    
        for card in cards:
                if card in deck:
                    deck.remove(card)
    for card in curr_trick:
        if card in deck:
            deck.remove(card)
        
    return hacks(hand, lead, curr_trick, deck, full_deck, player_no, trump, 
                 prev_tricks, deck_top, 0, phase_bids[player_no], player_data,
                 suppress_player_data)

def hacks(hand, lead, curr_trick, deck, full_deck, player_no, trump, 
          prev_tricks, deck_top, check_bid, bid, player_data, 
          suppress_player_data=True):
    """
    This function take many arguments
    hand - a tuple of strings, your hand
    lead - a string of the lead, unless it is None
    curr_trick - a tuple of strings
    deck - a list of strings, the cards remaining
    full_deck - a list of strings, the full_deck
    player_no - an integer of your player_no essentially
    trump - a string of the trump suit
    prev_tricks - the previous tricks as a tuple of tuples with strings
    deck_top - a string of the card to determine trump suit
    check_bid - an int (0 or 1), it is 1 if you are calling from bid
    and it is 0 if you want to make a play
    bid - your bid as an int
    player_data - the player_data which is the remaining deck
    Depending of check_bid, it will return either an int for bid
    or it will return a string for play.
    """
    import random
    from collections import defaultdict as dd
    
    # I will show what each of these mean when they are used
    curr_trick = list(curr_trick)
    player_no_backup = player_no
    card_rank = dd(int)
    random_list = dd(int)
    curr_trick_copy = curr_trick.copy()
    scores_overall = (0, 0, 0, 0)
    deck_copy = deck.copy()
    # the players needing more cards, because they haven't played yet
    need_extra_cards = 3 - len(curr_trick)
    
    # this finds which index (an int) the program should make
    # when finding your score from score_phase
    # eg 2 would be a score of 10 from (0,0,10,0)
    if len(prev_tricks) == 0:
        score_len = len(curr_trick)
    else:
        score_len = 0
        for prev_trick in prev_tricks:
            winner = best_card(prev_trick, trump, prev_trick[0][1])
            score_len += prev_trick.index(winner)
        score_len = (score_len + len(curr_trick))%4
        
        
    # this will run through samples of the game many times.    
    for j in range(0, 170):
    
        player_no = player_no_backup
        tricks = []
        curr_trick = curr_trick_copy
        deck = deck_copy.copy()
        count = 0
        opp1 = []
        opp2 = []
        opp3 = []
        hands = [list(hand)]
        
        # if I'm calling from bid, then create some opponents and add them
        # to a list of hands called hands.
        if check_bid == 1:
            for opp in [opp1, opp2, opp3]:
                    if len(deck) <= len(hand):
                        deck = full_deck.copy()
                    # for each opponent, get cards from deck then remove them
                    # from the deck
                    for i in range(len(hand)):
                        card = random.choice(deck)
                        opp.append(card)
                        deck.remove(card)
                    if hands.index(list(hand)) == player_no:
                        hands = hands + [opp]
                    else:
                        hands = [opp] + hands
        # does the same thing as if check_bid was 1, but it considers the 
        # players which need to have an extra card because they haven't made
        # their play yet.
        else:

            # for each opponent, gets cards from deck then remove them 
            # from the deck
            for opp in [opp1, opp2, opp3]:  
                if len(deck) <= len(hand):
                    deck = full_deck.copy()
                
                if count < need_extra_cards:
                    for i in range(len(hand)):
                        card = random.choice(deck)
                        opp.append(card)
                        deck.remove(card)
                    # count is used to control how many players gets that
                    # extra card
                    count += 1
                    hands = hands + [opp]
                else:
                    for i in range(len(hand)-1):
                        card = random.choice(deck)
                        opp.append(card)
                        deck.remove(card)
                    hands = [opp] + hands
        count = 0
        
        # this finishes the current trick if needed for play function 
        if check_bid == 0:
            player_no = len(curr_trick)
            while len(curr_trick) < 4:
                card = random.choice(validify(hands[player_no], curr_trick))
                # here we define the card we want to investigate
                # we run through the game as if you played this card first
                if check_bid == 0 and count == 0:
                    the_golden_card = card
                hands[player_no].remove(card)
                curr_trick = curr_trick + [card]
                player_no = (player_no + 1)%4
                count += 1

            tricks = [curr_trick.copy()]
            winner = best_card(curr_trick, trump, curr_trick[0][1])
     
            player_no = (player_no+curr_trick.index(winner))%4
        else:
            player_no = 0
            
        # this finishes the tricks until the players have no more cards
        while len(hands[0]) > 0:
            this_trick = []
            for i in range(4):
                # this card is a random card from the valid cards for
                # each player, it first validifies then makes a choice
                card = random.choice(validify(hands[player_no], this_trick))
                
                # the_golden_card will be the card that you play first
                # however, this is used if you are checking bid
                if check_bid == 1 and player_no == player_no_backup:
                    the_golden_card = card
                    
                this_trick.append(card)
                hands[player_no].remove(card)
                player_no = (player_no + 1)%4

            # this line is complicated
            # it finds the starting player for the next trick.
            # best_card first finds the winning card
            # it is indexed and then it has the same process as before
            player_no = (player_no+this_trick.index
                         (best_card(this_trick, trump, this_trick[0][1])))%4
            tricks.append(this_trick)
        
        # this is all the tricks
        tricks = list(prev_tricks) + tricks
        # this checks how many times each golden card was chosen, to normalise
        random_list[the_golden_card] += 1
        # card_rank (dict) deteremines the scores achieved by each golden card
        # the score_phase does most of the work here.
        # score_len at the end represents your index, your score. 
        # the bidis your bid.
        card_rank[the_golden_card] += score_phase((bid, bid, bid, bid), 
                                                  tricks, deck_top)[score_len]
    # normalises the scores    
    for i in card_rank:
        card_rank[i] = card_rank[i]/random_list[i]  
    # sorting the chosen cards by the scores acheieved
    card_rank = sorted(card_rank.items(), key=lambda x: x[1], reverse=True)
    
    # if you are looking for the play, then make the play that gives best
    # scores. However, if you are looking for bid then continue on.
    if check_bid == 0:
        if not suppress_player_data:
            return (card_rank[0][0], deck_copy)
        else:
            return card_rank[0][0]
    else:
        the_golden_card = card_rank[0][0]
        

    scores_overall = (0, 0, 0, 0)
    my_bid = 0
    
    # this is almost the same as the huge block of code before
    # the difference is that it assumes you will play the card from 
    # card_rank[0][0], the best card to play
    # and then runs through the phase and checks how many tricks you win.
    
    # reason I do this is because it integrates two strategies together
    # thus giving a more even result.
    for j in range(0, 110):
        player_no = player_no_backup
        tricks = []
        curr_trick = curr_trick_copy
        deck = deck_copy.copy()
        count = 0
        opp1 = []
        opp2 = []
        opp3 = []
        hands = [list(hand)]
        
        # creates opponents
        for opp in [opp1, opp2, opp3]:
            if len(deck) <= len(hand):
                deck = full_deck.copy()
            if check_bid == 1:
                for i in range(len(hand)):
                    card = random.choice(deck)
                    opp.append(card)
                    deck.remove(card)
                if hands.index(list(hand)) == player_no:
                    hands = hands + [opp]
                else:
                    hands = [opp] + hands
                    
        count = 0
        # creating the tricks
        while len(hands[0]) > 0:
            this_trick = []
            for i in range(4):
                 
                card = random.choice(validify(hands[player_no], this_trick))
                if player_no == player_no_backup and count == 0:
                    # in the first trick you must play the golden card
                    card = the_golden_card
                    
                this_trick.append(card)
                hands[player_no].remove(card)
                player_no = (player_no + 1)%4
                count += 1
            player_no = (player_no+this_trick.index
                         (best_card(this_trick, trump, this_trick[0][1])))%4
            tricks.append(this_trick)                  
        tricks = list(prev_tricks) + tricks
        # here we have scores_overall which looks at the addition
        # of the scores from each iteration.
        scores_overall = [sum(x) for x in zip
                          (score_phase((bid, bid, bid, bid), tricks, deck_top),
                           scores_overall)]
        
    # normalise the bid (the iteration is 110 times)
    my_bid = scores_overall[score_len]/110
    # rounding up or down
    if my_bid%1 > 0.5:
        my_bid += 1
    my_bid = my_bid//1
    
    # checks if you want to pass player_data, 
    if not suppress_player_data:
        return (int(my_bid), deck_copy)
    else:
        return (int(my_bid))
def validify(hand, curr_trick):
    """This function simply takes a hand (a tuple) and outputs the valid
    plays you can make in a list.
    """
    hand2 = hand.copy()
    hand3 = hand.copy()
    # go through the hand and remove invalid cards
    for card in hand2:
        if not is_valid_play(card, curr_trick, hand2):
            hand3.remove(card)
    return hand3
