<template>
    <AuthenticatedLayout>

        <!-- Game board -->
        <menu class="grid grid-cols-3 gap-1.5 w-0 min-w-fit mx-auto mt-12">
            <li v-for="(square, index) in boardState" class="bg-gray-300 size-24 grid place-items-center">
                <button v-if="square === 0" @click="fillSquare(index)"
                    class="place-self-stretch bg-gray-200 hover:bg-gray-300 transition-colors"
                    :disabled="!yourTurn"                    
                    ></button>
                <span v-else v-text="square === -1 ? 'X' : 'O'" class="text-4xl font-bold"> </span>
            </li>
        </menu>

        <!-- show active user -->

        <ul class="max-w-sm mx-auto mt-6 space-y-4">
            <li class="flex items-center gap-2">
                <span :class="{ 'bg-green-300': xTurn }" class="p-1.5 font-bold rounded bg-gray-200">X</span>
                <span>{{ game.player_one.name }}</span>
                <span :class="{ '!bg-green-500': players.find(({ id }) => id === game.player_one_id) }"
                    class="bg-red-500 size-2 rounded"></span>
            </li>

            <!-- player two -->
            <li class="flex items-center gap-2" v-if="game.player_two">
                <span :class="{ 'bg-green-300': !xTurn }" class="p-1.5 font-bold rounded bg-gray-200">Y</span>
                <span>{{ game.player_two.name }}</span>
                <span :class="{ '!bg-green-500': players.find(({ id }) => id === game.player_two_id) }"
                    class="bg-red-500 size-2 rounded"></span>
            </li>

            <li v-else>Waiting for player two...</li>
        </ul>

        <!-- show modalpop up when game state changes -->
        <Modal :show="gameState.hasEnded()" @close="resetGame()">
            <div class="p-6">

                <!-- // show the winner content -->
                <div class="text-6xl font-bold text-center my-12 font-mono uppercase">
                    <span v-if="gameState.current() === gameStates.OWins" class="text-green-600">O has won!</span>
                    <span v-else-if="gameState.current() === gameStates.XWins" class="text-green-600">X has won!</span>
                    <span v-else class="text-orange-600">Stalemate!</span>
                </div>

                <!-- //play again -->
                <div class="mt-3 flex justify-end">
                    <PrimaryButton @click="resetGame()">Play Again</PrimaryButton>
                </div>
            </div>
        </Modal>

    </AuthenticatedLayout>
</template>
<script setup>
import AuthenticatedLayout from '@/Layouts/AuthenticatedLayout.vue';
import { ref, computed, onUnmounted, onMounted } from 'vue';
import { useGameState, gameStates } from '@/Composables/useGameState.js';
import Modal from '@/Components/Modal.vue';
import PrimaryButton from '@/Components/PrimaryButton.vue';
import { router, usePage } from '@inertiajs/vue3';

const page = usePage();

const gameState = useGameState();

const props = defineProps({
    game: Object
})


// Game Board Rules
// -1 represent 'X'
// 0 represent 'space | empty'
// 1 represent 'O'

// show game state from db if it has, else set it to default state. 

const boardState = ref(props.game.state ?? [0, 0, 0, 0, 0, 0, 0, 0, 0]);

const players = ref([]);

const xTurn = computed(() => boardState.value.reduce((carry, value) => carry + value, 0) === 0);

const yourTurn = computed(() => {
    if (page.props.auth.user.id === props.game.player_one.id) {
        return xTurn.value;
    }

    return !xTurn.value;
})

const lines = [
    //rows
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],

    //columns
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],

    //diagonals
    [0, 4, 8],
    [2, 4, 6]
]

const channel = window.Echo.join(`games.${props.game.id}`)
    .here((users) => players.value = users)
    .joining((user) => router.reload({
        onSuccess: () => players.value.push(user)
    }))
    .leaving((user) => players.value = players.value.filter(({ id }) => id !== user.id))
    .listenForWhisper('PlayerMadeMove', ({ state }) => {
        boardState.value = state;
        checkForVictory();
    });

const updateOpponent = () => {

    router.put(route('games.update', props.game.id), {
        state: boardState.value
    });

    channel.whisper('PlayerMadeMove', {
        state: boardState.value
    });
}

const fillSquare = (index) => {

    if (!yourTurn.value) {
        return;
    }

    boardState.value[index] = xTurn.value ? -1 : 1;

    updateOpponent();

    checkForVictory();
}

function checkForVictory() {
    const winningLines = lines.map((line) => line.reduce((carry, index) => carry + boardState.value[index], 0))
        .find(sum => Math.abs(sum) === 3);

    // handle O win
    if (winningLines === 3) {
        gameState.change(gameStates.OWins);
        return
    }

    // handle x win
    if (winningLines === -3) {
        gameState.change(gameStates.XWins);
        return
    }

    // handle match tie condition
    if (!boardState.value.includes(0)) {
        gameState.change(gameStates.Stalemate);
        return;
    }

    gameState.change(gameStates.InProgress);
}

const resetGame = () => {
    boardState.value.fill(0);
    gameState.change(gameStates.InProgress);

    //whisper and reset game state in database also
    updateOpponent();
}

onMounted(checkForVictory);

onUnmounted(() => {
    window.Echo.leave(`games.${props.game.id}`)
})
</script>
