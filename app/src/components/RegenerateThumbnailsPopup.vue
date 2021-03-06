<template>
    <div
        class="overlay"
        v-if="isVisible">
        <div class="popup">
            <icon
                name="blank-image"
                size="xl"
                primaryColor="color-8" />

            <div class="render-popup-text">
                <template v-if="qualityChanged">
                    Images quality have been changed.
                </template>

                <template v-if="!qualityChanged">
                    Your theme has been changed.
                </template>

                <br>
                We warmly recommend you regenerate thumbnails.
            </div>

            <progress-bar
                :color="progressColor"
                :progress="progress"
                :stopped="progressIsStopped"
                :message="message" />

            <div class="buttons">
                <p-button
                    v-if="!regenerateIsDone"
                    @click.native="regenerate"
                    :disabled="regeneratingThumbnails"
                    type="medium no-border-radius half-width">
                    Regenerate thumbnails
                </p-button>

                <p-button
                    v-if="!regenerateIsDone"
                    @click.native="skip"
                    :disabled="regeneratingThumbnails"
                    type="medium no-border-radius half-width cancel-popup">
                    Skip regeneration
                </p-button>

                <p-button
                    v-if="regenerateIsDone"
                    @click.native="skip"
                    :disabled="regeneratingThumbnails"
                    type="medium no-border-radius full-width">
                    OK
                </p-button>
            </div>
        </div>
    </div>
</template>

<script>
import { ipcRenderer } from 'electron';

export default {
    name: 'regenerate-thumbnails-popup',
    data () {
        return {
            isVisible: false,
            qualityChanged: false,
            message: '',
            progress: 0,
            progressColor: 'blue',
            progressIsStopped: false,
            regeneratingThumbnails: false,
            regenerateIsDone: false
        };
    },
    mounted () {
        this.$bus.$on('regenerate-thumbnails-display', (config) => {
            this.isVisible = true;
            this.qualityChanged = config.qualityChanged;
            this.message = '';
            this.progress = 0;
            this.progressColor = 'blue';
            this.progressIsStopped = false;
            this.regeneratingThumbnails = false;
            this.regenerateIsDone = false;
        });
    },
    methods: {
        skip () {
            this.isVisible = false;
            this.qualityChanged = false;
            this.message = '';
            this.progress = 0;
            this.progressColor = 'blue';
            this.progressIsStopped = false;
            this.regeneratingThumbnails = false;
            this.regenerateIsDone = false;
        },
        regenerate () {
            if (this.regeneratingThumbnails) {
                return;
            }

            this.regeneratingThumbnails = true;
            this.progressIsStopped = false;
            this.message = 'Regenerating thumbnails...';

            setTimeout(() => {
                ipcRenderer.send('app-site-regenerate-thumbnails', {
                    name: this.$store.state.currentSite.config.name
                });

                ipcRenderer.once('app-site-regenerate-thumbnails-error', (event, data) => {
                    this.progressColor = 'red';
                    this.progressIsStopped = true;
                    this.message = data.message;
                    this.regeneratingThumbnails = false;
                    this.regenerateIsDone = true;
                });

                ipcRenderer.on('app-site-regenerate-thumbnails-progress', (event, data) => {
                    this.progress = data.value;
                    this.message = 'Progress: ' + data.value + '%';
                });

                ipcRenderer.once('app-site-regenerate-thumbnails-success', (event, data) => {
                    this.progress = 100;
                    this.progressColor = 'green';
                    this.progressIsStopped = true;
                    this.message = 'All thumbnails have been created.';
                    this.regeneratingThumbnails = false;
                    this.regenerateIsDone = true;
                });
            }, 350);
        }
    },
    beforeDestroy: function() {
        this.$bus.$off('regenerate-thumbnails-display');
        ipcRenderer.removeAllListeners('app-site-regenerate-thumbnails-error');
        ipcRenderer.removeAllListeners('app-site-regenerate-thumbnails-progress');
        ipcRenderer.removeAllListeners('app-site-regenerate-thumbnails-success');
    }
}
</script>

<style lang="scss" scoped>
@import '../scss/variables.scss';
@import '../scss/popup-common.scss';

.overlay {
    z-index: 100006;
}

.popup {
    background-color: $color-10;
    border: none;
    border-radius: .6rem;
    display: inline-block;
    font-size: 1.6rem;
    font-weight: 400;
    left: 50%;
    overflow: hidden;
    padding: 4rem 4rem 6rem 4rem;
    position: absolute;
    text-align: center;
    top: 50%;
    transform: translateX(-50%) translateY(-50%);
    width: 60rem;
}

.message {
    color: $color-5;
    font-weight: 400;
    margin: 0;
    padding: 4rem;
    position: relative;
    text-align: left;

    &.text-centered {
        text-align: center;
    }
}

.buttons {
    display: flex;
    margin: 4rem -4rem -6rem -4rem;
    position: relative;
    text-align: center;
    top: 1px;
}
</style>
