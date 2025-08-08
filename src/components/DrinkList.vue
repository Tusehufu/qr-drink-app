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
        <!-- Frågemodal -->
        <div class="modal fade show d-block" tabindex="-1" v-if="showQuestionModal">
            <div class="modal-dialog">
                <div class="modal-content border">
                    <div class="modal-header">
                        <h5 class="modal-title">Fyllekontroll</h5>
                    </div>
                    <div class="modal-body">
                        <p>{{ currentQuestion?.question }}</p>
                        <div class="form-check" v-for="option in currentQuestion?.options" :key="option">
                            <input class="form-check-input"
                                   type="radio"
                                   :id="option"
                                   :value="option"
                                   v-model="selectedAnswer" />
                            <label class="form-check-label" :for="option">{{ option }}</label>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button class="btn btn-primary" @click="submitQuestionAnswer" :disabled="!selectedAnswer">Skicka svar</button>
                    </div>
                </div>
            </div>
        </div>
        <div class="modal fade show d-block" tabindex="-1" v-if="showNameModal">
            <div class="modal-dialog">
                <div class="modal-content border">
                    <div class="modal-header">
                        <h5 class="modal-title">Välkommen!</h5>
                    </div>
                    <div class="modal-body">
                        <input v-model="customerNameInput" class="form-control" placeholder="Ange ditt namn" />
                    </div>
                    <div class="modal-footer">
                        <button class="btn btn-primary" @click="confirmName">Fortsätt</button>
                    </div>
                </div>
            </div>
        </div>
        <!-- Beställningsmodal -->
        <div class="modal fade show d-block" tabindex="-1" v-if="showOrderModal">
            <div class="modal-dialog">
                <div class="modal-content border">
                    <div class="modal-header">
                        <h5 class="modal-title">Beställning</h5>
                        <button type="button" class="btn-close" @click="closeOrderModal"></button>
                    </div>
                    <div class="modal-body text-center">
                        <div v-if="!orderConfirmed && !orderFeedback">
                            <p>Skickar beställning...</p>
                        </div>
                        <div v-if="orderFeedback">
                            <p class="fw-bold text-danger">{{ orderFeedback }}</p>
                        </div>
                        <div v-else-if="orderConfirmed">
                            <p class="fw-bold">Beställning mottagen! 🥳</p>
                            <p>Glöm inte att lämna in ditt glas!</p>
                        </div>
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
    import { ref, onMounted, computed, nextTick } from 'vue'
    import { supabase } from '../lib/supabase'

    type Drink = {
        id: number
        name: string
        description: string
        image: string
        soldOut?: boolean
        spirit: string

    }
    const drinks = ref<Drink[]>([])
    async function loadDrinks() {
        const { data, error } = await supabase.from('drinks').select('*')
        if (error) {
            console.error('Fel vid hämtning av drinkar:', error.message)
            return
        }
        drinks.value = data
    }
    const showQuestionModal = ref(false)
    const selectedAnswer = ref('')
    const currentQuestion = ref<{ question: string; options: string[]; correctAnswer: string } | null>(null)

    const questions = [
        {
            question: "Du går norrut, sen vänster, sen vänster igen. Vilket håll går du nu?",
            options: ["Söder", "Väster", "Öster", "Norr"],
            correctAnswer: "Söder",
        },
        {
            question: "En drink kostar 49 kr. Vad kostar 3 drinkar?",
            options: ["147", "150", "148", "144"],
            correctAnswer: "147"
        },
        {
            question: "Vad är nästa tal i serien: 2, 4, 8, 16, ...?",
            options: ["18", "24", "32", "30"],
            correctAnswer: "32",
        },
        {
            question: "Om ett flygplan kraschar på gränsen mellan Sverige och Norge, var begraver man överlevande?",
            options: ["Sverige", "Norge", "På gränsen", "Ingenstans"],
            correctAnswer: "Ingenstans",
        },
        {
            question: "Hur många bokstäver finns i alfabetet?",
            options: ["28", "29", "26", "30"],
            correctAnswer: "29",
        },
        {
            question: "Vilken siffra kommer härnäst? 1, 1, 2, 3, 5, 8, ?",
            options: ["11", "13", "10", "14"],
            correctAnswer: "13", 
        },
        {
            question: "Vilken planet är närmast solen?",
            options: ["Venus", "Jorden", "Merkurius", "Mars"],
            correctAnswer: "Merkurius",
        },
        {
            question: "Hur många kanter har ett stoppskylt (standard i Sverige)?",
            options: ["6", "8", "7", "10"],
            correctAnswer: "8", 
        },
        {
            question: "Vad får du om du adderar alla siffror i årtalet 2025?",
            options: ["9", "7", "11", "10"],
            correctAnswer: "9", 
        },
        {
            question: "Pelles mamma har fyra söner. De heter Nord, Väst och Syd. Vad heter den fjärde?",
            options: ["Öst", "Norr", "Det står inte", "Pelle"],
            correctAnswer: "Pelle",
        },
        {
            question: "Om 5 maskiner tar 5 minuter att göra 5 produkter, hur lång tid tar det för 100 maskiner att göra 100 produkter?",
            options: ["5 minuter", "100 minuter", "10 minuter", "1 minut"],
            correctAnswer: "5 minuter",
        },
    ]

    const isAdmin = ref(false)
    const showAdminModal = ref(false)
    const adminPasswordInput = ref('')
    const adminPassword = '1337' // Ändra till vad du vill
    const showOrderModal = ref(false)
    const selectedDrink = ref<Drink | null>(null)
    const customerName = ref('')
    const orderConfirmed = ref(false)
    const customerNameInput = ref('')
    const showNameModal = ref(false)
    const orderFeedback = ref('')

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
        const storedName = localStorage.getItem('customerName')
        if (storedName) {
            customerName.value = storedName
        } else {
            showNameModal.value = true
        }

        loadDrinks()
        setInterval(() => {
            if (!isAdmin.value) loadDrinks()
        }, 5000)
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
    async function markSoldOut(drink: Drink) {
        const { error } = await supabase.from('drinks').update({ soldOut: true }).eq('id', drink.id)
        if (!error) await loadDrinks()
    }

    async function restoreDrink(drink: Drink) {
        const { error } = await supabase.from('drinks').update({ soldOut: false }).eq('id', drink.id)
        if (!error) await loadDrinks()
    }


    //function markSoldOut(drink: Drink) {
    //    drink.soldOut = true
    //}

    //function restoreDrink(drink: Drink) {
    //    drink.soldOut = false
    //}

    function orderDrink(drink: Drink) {
        if (drink.soldOut || !customerName.value) return
        selectedDrink.value = drink
        submitOrder()
    }


    function closeOrderModal() {
        showOrderModal.value = false
        selectedDrink.value = null
        orderConfirmed.value = false
    }
    function confirmName() {
        const trimmed = customerNameInput.value.trim()
        if (trimmed.length < 2) {
            alert('Vänligen fyll i ditt namn')
            return
        }
        customerName.value = trimmed
        localStorage.setItem('customerName', trimmed)
        showNameModal.value = false
    }

    async function submitQuestionAnswer() {
        if (!currentQuestion.value || !selectedAnswer.value) return

        if (selectedAnswer.value === currentQuestion.value.correctAnswer) {
            showQuestionModal.value = false
            orderConfirmed.value = true
            await finalizeOrder()
        } else {
            alert('Fel svar! Du är blockerad i 15 minuter.')
            localStorage.setItem('nextOrderTime', (Date.now() + 15 * 60 * 1000).toString())
            showQuestionModal.value = false
            closeOrderModal()
        }
    }
    async function finalizeOrder() {
        //const { error } = await supabase.from('orders').insert([
        //    {
        //        drink_id: selectedDrink.value!.id,
        //        drink_name: selectedDrink.value!.name,
        //        customer: customerName.value,
        //    },
        //])

        //if (error) {
        //    alert('Fel vid beställning: ' + error.message)
        //    return
        //}

        orderConfirmed.value = true
        // ⚠️ Endast sätt spärrtid om det inte redan finns en
        if (!localStorage.getItem('nextOrderTime')) {
            const THIRTY_MINUTES = 30 * 60 * 1000
            const futureBlockTime = Date.now() + THIRTY_MINUTES
            localStorage.setItem('nextOrderTime', futureBlockTime.toString())
        }

        setTimeout(() => {
            closeOrderModal()
        }, 3000)
    }

    async function submitOrder() {
        if (!selectedDrink.value || !customerName.value) return

        // Kontrollera blockering med återstående tid
        const blockedUntil = localStorage.getItem('nextOrderTime')
        if (blockedUntil && parseInt(blockedUntil) > Date.now()) {
            const minutesLeft = Math.ceil((parseInt(blockedUntil) - Date.now()) / 60000)
            orderFeedback.value = `Du är för full. Drick ett glas vatten och försök igen om ca ${minutesLeft} minut(er).`
            orderConfirmed.value = false

            showOrderModal.value = true
            await nextTick()

            setTimeout(() => {
                orderFeedback.value = ''
                closeOrderModal()
            }, 3000)
            return
        }

        // Hämta antal tidigare beställningar
        const { count, error: countError } = await supabase
            .from('orders')
            .select('*', { count: 'exact', head: true })
            .eq('customer', customerName.value)

        if (countError) {
            alert('Kunde inte kontrollera tidigare beställningar.')
            return
        }

        // Kontrollera tid för första beställning (30-minutersspärr)
        if (count !== null && count > 0) {
            const { data: firstOrder, error: firstOrderError } = await supabase
                .from('orders')
                .select('created_at')
                .eq('customer', customerName.value)
                .order('created_at', { ascending: true })
                .limit(1)

            if (firstOrderError || !firstOrder?.[0]) {
                console.warn('Kunde inte hämta första beställningens tid')
            } else {
                const firstOrderTime = new Date(firstOrder[0].created_at).getTime()
                const now = Date.now()
                const THIRTY_MINUTES = 30 * 60 * 1000

                if (now - firstOrderTime < THIRTY_MINUTES) {
                    const minutesLeft = Math.ceil((THIRTY_MINUTES - (now - firstOrderTime)) / 60000)
                    localStorage.setItem('nextOrderTime', (firstOrderTime + THIRTY_MINUTES).toString())

                    orderFeedback.value = `Du måste vänta ${minutesLeft} minut(er) innan du kan beställa igen.`
                    orderConfirmed.value = false
                    showOrderModal.value = true
                    await nextTick()

                    setTimeout(() => {
                        orderFeedback.value = ''
                        closeOrderModal()
                    }, 3000)
                    return
                }
            }
        }

        // Om fler än 1 tidigare beställning, visa fråga
        if (count !== null && count > 1) {
            currentQuestion.value = questions[Math.floor(Math.random() * questions.length)]
            selectedAnswer.value = ''
            showQuestionModal.value = true
            return
        }

        // Slutligen: skicka in beställningen
        showOrderModal.value = true
        await finalizeOrder()
    }





</script>
