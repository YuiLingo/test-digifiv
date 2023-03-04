<template>
    <div class="container">
        <div class="row align-items-start">
            <div class="col-3">
                <div class="card text-bg-info text-center">
                    <img src="https://icons-for-free.com/iconfiles/png/512/avatar+human+people+profile+user+icon-1320168139431219590.png"
                        class="card-img-top" alt="Player 1">
                    <div class="card-body">
                        <h5 class="card-title">Player 1</h5>
                        <p class="card-text">
                            Points: {{ matchDetails.points.hasOwnProperty(0) ? matchDetails.points[0] : 0 }}
                        </p>
                        <a v-on:click='drawCard(0)' class="btn btn-primary">DRAW</a>
                    </div>
                </div>
            </div>
            <div class="col-6">
                <div class="card d-flex justify-content-center border-dark text-dark mb-3">
                    <div class="card-header">
                        Round {{ matchDetails.round.number + 1 }}
                        <span class="text-muted">Cards left {{ matchDetails.cards.length }}</span>
                    </div>
                    <div class="card-body">
                        <div class="card-group">
                            <div class="card">
                                <img :src="matchDetails.preview[0].image" class="card-img-top" :alt="matchDetails.preview[0].label">
                                <div class="card-body">
                                    <h5 class="card-title">
                                        {{ matchDetails.preview[0].label }}
                                    </h5>
                                </div>
                            </div>
                            <div class="card">
                            <img :src="matchDetails.preview[1].image" class="card-img-top" :alt="matchDetails.preview[1].label">
                                <div class="card-body">
                                    <h5 class="card-title">
                                        {{ matchDetails.preview[1].label }}
                                    </h5>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="card-footer d-flex justify-content-center">
                        <button v-on:click="resetMatch()" class="btn btn-secondary">REMATCH</button>
                    </div>
                </div>
            </div>
            <div class="col-3">
                <div class="card text-bg-warning text-center">
                    <img src="https://cdn3.iconfinder.com/data/icons/vector-icons-6/96/256-512.png" class="card-img-top"
                        alt="Player 2">
                    <div class="card-body">
                        <h5 class="card-title">Player 2</h5>
                        <p class="card-text">
                            Points: {{ matchDetails.points.hasOwnProperty(1) ? matchDetails.points[1] : 0 }}
                        </p>
                        <a v-on:click='drawCard(1)' class="btn btn-danger">DRAW</a>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            matchDetails: {
                histories: [], // array foreach round [ { type:, number: },, ]
                cards: [], // cards left
                round: {
                    number: 0,
                    result: [] // [ { type:, number:},, ]
                },
                points: [],
                preview: [
                    {
                        image: '',
                        label: '',
                    },
                    {
                        image: '',
                        label: '',
                    }
                ],
            },
            card: {
                type: [
                    'diamonds',
                    'clubs',
                    'hearts',
                    'spades',
                ],
                number: [
                    'ace',
                    2,
                    3,
                    4,
                    5,
                    6,
                    7,
                    8,
                    9,
                    10,
                    'jack',
                    'queen',
                    'king',
                ]
            }
        }
    },
    methods: {
        resetMatch() {
            let currentIntance = this;
            currentIntance.matchDetails = {
                histories: [],
                cards: [],
                round: {
                    number: 0,
                    result: []
                },
                points: [],
                preview: [
                    {
                        image: '',
                        label: '',
                    },
                    {
                        image: '',
                        label: '',
                    }
                ]
            };

            currentIntance.card.type.forEach(cardType => {
                currentIntance.card.number.forEach(number => {
                    currentIntance.matchDetails.cards.push({
                        type: cardType,
                        number: number
                    })
                })
            });

            currentIntance.matchDetails.cards = this.shuffleArray(currentIntance.matchDetails.cards);
        },
        resetRound(next = true) {
            let currentIntance = this;
            next ? currentIntance.matchDetails.round.number++ : currentIntance.matchDetails.round.number = 0;
            currentIntance.matchDetails.round.result = [];
            currentIntance.matchDetails.preview = [
                {
                    image: '',
                    label: '',
                },
                {
                    image: '',
                    label: '',
                }
            ];
        },
        drawCard(player) {

            let currentIntance = this;

            if (player > 2) {
                return Swal.fire({
                    icon: 'error',
                    title: 'Oops...',
                    text: 'Something went wrong!',
                });
            }

            if (currentIntance.matchDetails.round.result.filter(function (result) { return result }).length >= 2) {
                currentIntance.resetRound();
            }

            if (currentIntance.matchDetails.round.result.hasOwnProperty(player)) {
                return Swal.fire({
                    icon: 'warning',
                    title: 'Be patient...',
                    text: 'Player already drawed for this round',
                });
            }

            if (currentIntance.matchDetails.cards.length <= 0) {
                return Swal.fire({
                    icon: 'info',
                    title: 'Match Finished',
                    text: 'Cards have been drawed out, please press REMATCH to play again.',
                });
            }

            let drawedCard = currentIntance.matchDetails.cards.pop();
            currentIntance.matchDetails.round.result[player] = drawedCard;

            let image = '/assets/' + drawedCard.type + '_' + drawedCard.number + '.svg';

            currentIntance.matchDetails.preview[player].label = drawedCard.type + ' ' + drawedCard.number;
            if (currentIntance.imageExists(image)) {
                currentIntance.matchDetails.preview[player].image = image;
            } else {
                currentIntance.matchDetails.preview[player].image = '/assets/placeholder.svg';
            }
            console.log(currentIntance.matchDetails.cards);
            console.log(currentIntance.matchDetails.round);

            // Each Round Ended
            if (currentIntance.matchDetails.round.result.filter(function (result) { return result }).length >= 2) {

                currentIntance.matchDetails.histories[currentIntance.matchDetails.round.number] = currentIntance.matchDetails.round.result;

                let winnerIndex = currentIntance.matchCard(currentIntance.matchDetails.round.result);

                // initial index
                if (!currentIntance.matchDetails.points.hasOwnProperty(winnerIndex)) {
                    currentIntance.matchDetails.points[winnerIndex] = 0;
                }
                currentIntance.matchDetails.points[winnerIndex]++;

                // When ALl Card Drawed
                if (currentIntance.matchDetails.cards.length <= 0) {

                    let playerPoints = [];

                    currentIntance.matchDetails.histories.forEach(function (result) {
                        let winnerIndex = currentIntance.matchCard(result);

                        // initial index
                        if (!playerPoints.hasOwnProperty(winnerIndex)) {
                            playerPoints[winnerIndex] = 0;
                        }
                        playerPoints[winnerIndex]++;

                        let largestPlayersIndex = currentIntance.calculatePoints(playerPoints);
                        let title = '';

                        if (largestPlayersIndex.length > 1) {
                            largestPlayersIndex.forEach(function (playerIndex, index) {
                                title += `${(index ? 'and ' : '')}Player ${playerIndex+1} `;
                            });
                            title += `have the same (${playerPoints[largestPlayersIndex[0]]}) winning point.`
                        } else {
                            title = `Player ${largestPlayersIndex[0]+1} have the most (${playerPoints[largestPlayersIndex[0]]}) winning point.`;
                        }

                        Swal.fire({
                            title: title,
                            backdrop: '#00000066'
                        });
                    });
                } else {

                    Swal.fire({
                        title: `Player ${winnerIndex+1} win 1 point`,
                        backdrop: '#00000066'
                    });
                }
            }
        },
        matchCard(result) {
            let largerIndex = -1;

            for (let i = 1; i < result.length; i++) {

                if (this.card.number.indexOf(result[i].number) > this.card.number.indexOf(result[i - 1].number)) {
                    // Check Current Number > Previous Number
                    largerIndex = i;
                } else if (this.card.number.indexOf(result[i].number) == this.card.number.indexOf(result[i - 1].number)) {
                    // Check Current Number = Previous Number
                    if (this.card.type.indexOf(result[i].type) > this.card.type.indexOf(result[i - 1].type)) {
                        // Check Current Type > Previous Type
                         largerIndex = i;
                    } else {
                         largerIndex = i - 1;
                    }

                } else {
                    // Check Current Number = Previous Number
                    largerIndex = i - 1;
                }
            }
            return largerIndex;
        },
        calculatePoints(playerPoints) {
            let largest = playerPoints[0];
            let largestIndex = [0];
            for (let i = 1; i < playerPoints.length; i++) {
                if (playerPoints[i] > largest) {
                    largest = playerPoints[i];
                    largestIndex = [i];
                } else if (playerPoints[i] === largest) {
                    largestIndex.push(i);
                }
            }
            return largestIndex;
        },
        imageExists(imageUrl) {

            let http = new XMLHttpRequest();

            http.open('HEAD', imageUrl, false);
            http.send();

            return http.status != 404;

        },
        shuffleArray(array) {
            const len = array.length;
            for (let i = len - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [array[i], array[j]] = [array[j], array[i]];
            }
            return array;
        }
    },
    mounted() {
        // methods can be called in lifecycle hooks, or other methods!
        this.resetMatch();
    }
}
</script>

<style scoped>
    .card .card-img-top {
        padding: 1rem;
    }

    .card .card-group {
        min-height: 350px;
    }
</style>