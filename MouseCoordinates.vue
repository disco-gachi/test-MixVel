<script setup lang="ts">
import { useDebounceFn } from '@vueuse/core';

interface ChannelMessage {
    coordinates: Coordinates;
}
interface Coordinates {
    x: number;
    y: number;
}

const channel = ref<BroadcastChannel>(
    createChannel()
);
const coordinates = ref<Coordinates>({
    x: 0,
    y: 0,
});

function createChannel () {
    return new BroadcastChannel('mouse_channel');
}

function startListenChannel (channel: Ref<BroadcastChannel>) {
    channel.value.addEventListener('message', onChannelMessage);
}

function onChannelMessage ({ data: { coordinates } }: MessageEvent<ChannelMessage>) {
    console.log('coords received', coordinates);
    updateCoordinates(coordinates);
}


function startListenMouseMove () {
    document.addEventListener('mousemove', onMouseMove);
}

const postChannelMessageDebounced = useDebounceFn(postChannelMessage, 100);

function onMouseMove (event: MouseEvent) {
    const newCoordinates = makeCoordinates(event);

    updateCoordinates(newCoordinates);
    postChannelMessageDebounced(
        makeChannelMessage(newCoordinates)
    );
}

function updateCoordinates (update: Coordinates) {
    coordinates.value = { ...update };
}

function postChannelMessage (message: ChannelMessage) {
    try {
        channel.value.postMessage(message);
    } catch (e: unknown) {
        // reconnect here...
    }
}

function makeChannelMessage (coordinates: Coordinates): ChannelMessage {
    return {
        coordinates,
    };
}

function makeCoordinates (event: MouseEvent): Coordinates {
    return {
        x: event.pageX,
        y: event.pageY,
    };
}

function init () {
    startListenChannel(channel);
    startListenMouseMove();
}

function destroy () {
    channel.value.close();
    document.removeEventListener('mousemove', onMouseMove);
}

onMounted(init);
onBeforeUnmount(destroy);
</script>

<template>
    <div>
        <div>
            <span>X:</span> {{ coordinates.x }}
        </div>
        <div>
            <span>Y:</span> {{ coordinates.y }}
        </div>
    </div>
</template>
