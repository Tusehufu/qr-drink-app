<template>
    <div>
        <button class="btn btn-outline-secondary position-fixed bottom-0 end-0 m-3 z-3"
                @click="showAdminModal = true">
            Admin
        </button>
        <div class="modal fade show d-block" tabindex="-1" role="dialog" v-if="showAdminModal">
            <div class="modal-dialog" role="document">
                <div class="modal-content border">
                    <div class="modal-header">
                        <h5 class="modal-title">Ange adminlösenord</h5>
                        <button type="button" class="btn-close" @click="showAdminModal = false"></button>
                    </div>
                    <div class="modal-body">
                        <input v-model="adminPasswordInput" type="password" class="form-control" placeholder="Lösenord" />
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" @click="showAdminModal = false">Stäng</button>
                        <button type="button" class="btn btn-primary" @click="verifyAdmin">Logga in</button>
                    </div>
                </div>
            </div>
        </div>
        <!-- Beställningsmodal -->
        <div class="modal fade show d-block" tabindex="-1" v-if="showOrderModal">
            <div class="modal-dialog">
                <div class="modal-content border">
                    <div class="modal-header">
                        <h5 class="modal-title">Ange ditt namn</h5>
                        <button type="button" class="btn-close" @click="closeOrderModal()"></button>
                    </div>
                    <div class="modal-body">
                        <div v-if="!orderConfirmed">
                            <input v-model="customerName" class="form-control" placeholder="Ditt namn" />
                        </div>
                        <div v-else class="text-center">
                            <p class="fw-bold">Beställning mottagen! 🥳</p>
                            <p>Glöm inte att lämna in ditt glas!</p>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button class="btn btn-secondary" @click="closeOrderModal()">Avbryt</button>
                        <button class="btn btn-primary" @click="submitOrder()">Skicka beställning</button>
                    </div>
                </div>
            </div>
        </div>
        <div v-for="(drinkList, spirit) in groupedDrinks" :key="spirit" class="mb-4">
            <h4 class="text-primary mb-3">{{ spirit }}</h4>
            <div class="row">
                <div class="col-md-6 col-lg-4 mb-4" v-for="drink in drinkList" :key="drink.id">
                    <div class="card h-100 shadow-sm">
                        <div class="d-flex flex-row h-100">
                            <!-- Textkolumn -->
                            <div class="p-3 d-flex flex-column justify-content-between flex-grow-1">
                                <div>
                                    <h5 class="card-title">{{ drink.name }}</h5>
                                    <p class="card-text">{{ drink.description }}</p>
                                </div>
                                <button class="btn btn-primary mt-3 align-self-start"
                                        :disabled="drink.soldOut"
                                        @click="orderDrink(drink)">
                                    {{ drink.soldOut ? 'Slutsåld' : 'Beställ' }}
                                </button>
                                <!-- Admin-knapp -->
                                <button v-if="isAdmin && !drink.soldOut"
                                        class="btn btn-sm btn-outline-danger mt-2 align-self-start"
                                        @click="markSoldOut(drink)">
                                    Ta bort
                                </button>
                                <button v-if="isAdmin && drink.soldOut"
                                        class="btn btn-sm btn-outline-success mt-2 align-self-start"
                                        @click="restoreDrink(drink)">
                                    Återställ
                                </button>
                            </div>

                            <!-- Bildkolumn -->
                            <img :src="drink.image"
                                 class="img-fluid rounded-end"
                                 alt="Bild på {{ drink.name }}"
                                 style="object-fit: cover; width: 120px; height: 100%;" />
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>


<script setup lang="ts">
    import { ref, onMounted, computed } from 'vue'
    import { supabase } from '../lib/supabase'

    type Drink = {
        id: number
        name: string
        description: string
        image: string
        soldOut?: boolean
        spirit: String

    }

    const drinks = ref<Drink[]>([
        {
            id: 1,
            name: 'The Bicycle Thief',
            description: 'Gin, Campari, Grapefruktjuice, Citronjuice, Sockerlag, Sodavatten',
            image: '/images/the-bicycle-thief.jpg',
            spirit: 'Gin',
        },
        {
            id: 2,
            name: 'Pornstar Martini',
            description: 'Vodka, Passionsfruktslikör, Sockerlag, Limejuice',
            image: '/images/pornstar-martini.jpg',
            spirit: 'Vodka',
        },
        {
            id: 3,
            name: 'Gin Fizz',
            description: 'Gin, citronjuice, sockerlag, äggvita, sodavatten',
            image: '/images/gin-fizz.jpg',
            spirit: 'Gin',
        },
        {
            id: 4,
            name: 'Spicy Margarita',
            description: 'Tequila, Triple sec, Limejuice, AgaveSirap',
            image: '/images/spicy-margarita.jpg',
            spirit: 'Tequila',
        },
        {
            id: 5,
            name: 'Fizzypop',
            description: 'Vodka, Blå Curaçao, Monin BubblegumSirap, Citronjuice, 7up',
            image: '/images/fizzy-pop.jpg',
            spirit: 'Vodka',
        },
        {
            id: 6,
            name: 'Midori Sour',
            description: 'Vodka, Midori, Citronjuice, Limejuice, Sodavatten',
            image: '/images/midori-sour.jpg',
            spirit: 'Vodka',
        },
        {
            id: 7,
            name: 'Tom Collins',
            description: 'Gin, Citronjuice, Sockerlag, Sodavatten',
            image: '/images/tom-collins.jpg',
            spirit: 'Gin',
        },
        {
            id: 8,
            name: 'Margarita',
            description: 'Tequila, Triple sec, Limejuice, Sockerlag',
            image: '/images/margarita.jpg',
            spirit: 'Tequila',
        },
        {
            id: 9,
            name: 'Caipirinha',
            description: 'Cachaça, Lime, Socker',
            image: '/images/caipirinha.jpg',
            spirit: 'Rom',
        },
        {
            id: 10,
            name: 'Kiwi Caipirinha',
            description: 'Cachaça, Lime, Socker, Kiwi',
            image: '/images/kiwi-caipirinha.jpg',
            spirit: 'Rom',
        },
        {
            id: 11,
            name: 'Aperol Spritz',
            description: 'Aperol, Bubbel, Sodavatten',
            image: '/images/aperol-spritz.jpg',
            spirit: 'Aperol',
        },
        {
            id: 12,
            name: 'Thai Basil',
            description: 'Gin, Citronjuice, Thaibasilikasockerlag, Kokosskum',
            image: '/images/thai-basil.jpg',
            spirit: 'Gin',
        },
        {
            id: 13,
            name: 'Southside',
            description: 'Gin, Limejuice, Sockerlag, Mynta',
            image: '/images/south-side.jpg',
            spirit: 'Gin',
        },
        {
            id: 14,
            name: 'Mojito',
            description: 'Rom, Limejuice, Sockerlag, Sodavatten, Mynta',
            image: '/images/mojito.jpg',
            spirit: 'Rom',
        },
    ])
    const isAdmin = ref(false)
    const showAdminModal = ref(false)
    const adminPasswordInput = ref('')
    const adminPassword = '1337' // Ändra till vad du vill
    const showOrderModal = ref(false)
    const selectedDrink = ref<Drink | null>(null)
    const customerName = ref('')
    const orderConfirmed = ref(false)

    const groupedDrinks = computed(() => {
        const groups: Record<string, Drink[]> = {}
        for (const drink of drinks.value) {
            const category = drink.spirit
            if (!groups[category]) {
                groups[category] = []
            }
            groups[category].push(drink)
        }
        return groups
    })

    onMounted(() => {
        isAdmin.value = localStorage.getItem('isAdmin') === 'true'
    })

    function verifyAdmin() {
        if (adminPasswordInput.value === adminPassword) {
            isAdmin.value = true
            localStorage.setItem('isAdmin', 'true')
            showAdminModal.value = false
            adminPasswordInput.value = ''
        } else {
            alert('Fel lösenord')
        }
    }

    function markSoldOut(drink: Drink) {
        drink.soldOut = true
    }

    function restoreDrink(drink: Drink) {
        drink.soldOut = false
    }

    function orderDrink(drink: Drink) {
        if (drink.soldOut) return
        selectedDrink.value = drink
        customerName.value = ''
        showOrderModal.value = true
    }

    function closeOrderModal() {
        showOrderModal.value = false
        selectedDrink.value = null
        customerName.value = ''
        orderConfirmed.value = false
    }


    async function submitOrder() {
        if (!selectedDrink.value || customerName.value.trim() === '') {
            alert('Vänligen fyll i ditt namn')
            return
        }

        const { error } = await supabase.from('orders').insert([
            {
                drink_id: selectedDrink.value.id,
                drink_name: selectedDrink.value.name,
                customer: customerName.value.trim(),
            },
        ])

        if (error) {
            alert('Fel vid beställning: ' + error.message)
            return
        }

        orderConfirmed.value = true

        // Vänta 3 sekunder, stäng modalen
        setTimeout(() => {
            closeOrderModal()
        }, 3000)
    }


</script>
