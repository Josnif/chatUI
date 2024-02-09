<template>
    <div class="overlay-menu">
        <ul>
            <li v-for="item in menuItems" :key="item.title" @click="item.action">
                <span :class="['fas', item.icon, 'menu-item-icon']"></span>
                <span class="title">{{ item.title }}</span>
            </li>
        </ul>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                menuItems: [
                    { title: 'Datenschutz', icon: 'fa-shield-alt', action: this.onPrivacy },
                    { title: 'Impressum', icon: 'fa-file-alt', action: this.onLegalNotice },
                    { title: 'Infomaterial', icon: 'fa-info-circle', action: this.onInfoMaterial },
                    { title: 'Bewerbung', icon: 'fa-graduation-cap', action: this.onApplication },
                    { title: 'Infotermine', icon: 'fa-calendar-alt', action: this.onInfoDates }                    
                ]
            };
        },
        methods: {
            onPrivacy() { window.open('https://www.amdnet.de/datenschutz/', '_blank'); },
            onLegalNotice() { window.open('https://www.amdnet.de/impressum/', '_blank'); },
            onInfoMaterial() { window.open('https://www.amdnet.de/infomaterial/', '_blank'); },
            onApplication() { window.open('https://www.amdnet.de/bewerbung/', '_blank'); },
            onInfoDates() { window.open('https://www.amdnet.de/termine/', '_blank'); },            
        }
    };
</script>
