<script setup lang="ts">
import { computed, ref } from "vue";

// Einstellungen
const passwordLength = ref(16);
const useUppercase = ref(true);
const useLowercase = ref(true);
const useNumbers = ref(true);
const useSymbols = ref(true);
const generatedPassword = ref("");
const isCopied = ref(false);

// Passwort generieren
function generatePassword() {
  const charSets = {
    uppercase: "ABCDEFGHIJKLMNOPQRSTUVWXYZ",
    lowercase: "abcdefghijklmnopqrstuvwxyz",
    numbers: "0123456789",
    symbols: "!@#$%^&*()_+-=[]{}|;:,.<>?/",
  };

  let allowedCharacters = "";
  if (useUppercase.value) allowedCharacters += charSets.uppercase;
  if (useLowercase.value) allowedCharacters += charSets.lowercase;
  if (useNumbers.value) allowedCharacters += charSets.numbers;
  if (useSymbols.value) allowedCharacters += charSets.symbols;
  if (!allowedCharacters) return;

  let result = "";
  for (let i = 0; i < passwordLength.value; i++) {
    result += allowedCharacters[Math.floor(Math.random() * allowedCharacters.length)];
  }
  generatedPassword.value = result;
}

// In Zwischenablage kopieren
async function copyToClipboard() {
  if (!generatedPassword.value) return;
  try {
    await navigator.clipboard.writeText(generatedPassword.value);
    isCopied.value = true;
    setTimeout(() => {
      isCopied.value = false;
    }, 2000);
  } catch (err) {
    console.error("Kopieren fehlgeschlagen:", err);
  }
}

// ──────────────────────────────────────────
// Entropie-Berechnung
// Formel: Bits = Länge × log₂(Zeichenpool)
// Je mehr Bits → desto schwerer zu knacken
// ──────────────────────────────────────────

const poolSize = computed(() => {
  let size = 0;
  if (useUppercase.value) size += 26;
  if (useLowercase.value) size += 26;
  if (useNumbers.value) size += 10;
  if (useSymbols.value) size += 27;
  return size;
});

const entropyBits = computed(() => {
  if (poolSize.value === 0) return 0;
  return passwordLength.value * Math.log2(poolSize.value);
});

const BarWidth = computed(() => Math.min((entropyBits.value / 196) * 100, 100));

const strengthLabel = computed(() => {
  if (entropyBits.value < 40) return "Schwach";
  if (entropyBits.value < 60) return "Okay";
  if (entropyBits.value < 80) return "Stark";
  return "Sehr Stark";
});

const strengthColor = computed(() => {
  if (entropyBits.value < 40) return "#f87171";
  if (entropyBits.value < 60) return "#fbbf24";
  if (entropyBits.value < 80) return "#60a5fa";
  return "#34d399";
});

// Cracking-Zeit: wie lange bräuchte ein Angreifer mit 1 Mrd. Versuche/Sek.
const estimatedCrackTime = computed(() => {
  const secondsNeeded = Math.pow(2, entropyBits.value) / 1_000_000_000 / 2;
  if (secondsNeeded < 60) return "< 1 Minute";
  if (secondsNeeded < 3_600) return `~${Math.round(secondsNeeded / 60)} Minuten`;
  if (secondsNeeded < 86_400) return `~${Math.round(secondsNeeded / 3_600)} Stunden`;
  if (secondsNeeded < 31_536_000) return `~${Math.round(secondsNeeded / 86_400)} Tage`;
  if (secondsNeeded < 31_536_000 * 1_000) return `~${Math.round(secondsNeeded / 31_536_000)} Jahre`;
  if (secondsNeeded < 31_536_000 * 1_000_000) return `~${(secondsNeeded / 31_536_000 / 1_000).toFixed(0)}K Jahre`;
  return "Praktisch unknackbar";
});
</script>

<template>
  <div class="min-h-screen bg-[#0d0e1a] flex flex-col items-center justify-center px-8 py-16 relative overflow-hidden">
    <!-- Hintergrund-Orbs -->
    <div
      class="absolute top-[-200px] left-[-200px] w-[500px] h-[500px] bg-indigo-600 rounded-full opacity-[0.08] blur-[120px] pointer-events-none"
    ></div>
    <div
      class="absolute bottom-[-200px] right-[-150px] w-[500px] h-[500px] bg-violet-700 rounded-full opacity-[0.08] blur-[120px] pointer-events-none"
    ></div>

    <!-- Header -->
    <div class="text-center mb-12">
      <h1 class="text-4xl font-bold text-white tracking-tight mb-2">Password Generator</h1>
      <p class="text-xs text-indigo-400/70 uppercase tracking-[3px] font-medium">Cryptographically Secure</p>
    </div>

    <!-- Hauptcontainer -->
    <div
      class="w-full max-w-5xl bg-white/[0.03] border border-white/[0.06] rounded-3xl backdrop-blur-xl overflow-hidden shadow-2xl"
    >
      <div class="grid grid-cols-[1.1fr_0.9fr] divide-x divide-white/[0.06]">
        <!-- Linke Seite: Generator -->
        <div class="p-10">
          <div class="flex gap-3 mb-8">
            <input
              type="text"
              v-model="generatedPassword"
              placeholder="Your password appears here..."
              class="flex-1 bg-white/[0.05] border border-white/[0.08] rounded-xl px-5 py-4 text-indigo-100 font-mono text-sm outline-none focus:border-indigo-500/40 transition placeholder-white/[0.15]"
            />
            <button
              @click="copyToClipboard"
              :class="isCopied ? 'bg-emerald-600 hover:bg-emerald-500' : 'bg-indigo-600 hover:bg-indigo-500'"
              class="text-white font-semibold text-sm px-7 py-4 rounded-xl transition-all duration-200 whitespace-nowrap"
            >
              {{ isCopied ? "✓ Copied" : "Copy" }}
            </button>
          </div>

          <!-- Länge + Balken -->
          <div class="mb-8">
            <div class="flex justify-between items-center mb-3">
              <span class="text-white/40 text-xs font-semibold uppercase tracking-widest">Password Length</span>
              <span class="text-indigo-300 font-mono text-lg font-medium">{{ passwordLength }}</span>
            </div>
            <input type="range" min="6" max="32" v-model="passwordLength" class="w-full accent-indigo-500 mb-4" />

            <div class="w-full h-1.5 bg-white/[0.08] rounded-full overflow-hidden mb-2.5">
              <div
                class="h-full rounded-full transition-all duration-300"
                :style="{ width: BarWidth + '%', background: strengthColor }"
              />
            </div>
            <div class="flex justify-between items-center">
              <span class="text-xs font-semibold" :style="{ color: strengthColor }">● {{ strengthLabel }}</span>
              <span class="text-white/20 text-xs font-mono">{{ entropyBits.toFixed(1) }} bits</span>
            </div>
          </div>

          <!-- Zeichentypen -->
          <div class="grid grid-cols-2 gap-3 mb-8">
            <label class="flex items-center gap-3 text-white/50 text-sm cursor-pointer hover:text-white/80 transition">
              <input type="checkbox" v-model="useUppercase" class="accent-indigo-500 w-4 h-4" />
              Uppercase
            </label>
            <label class="flex items-center gap-3 text-white/50 text-sm cursor-pointer hover:text-white/80 transition">
              <input type="checkbox" v-model="useLowercase" class="accent-indigo-500 w-4 h-4" />
              Lowercase
            </label>
            <label class="flex items-center gap-3 text-white/50 text-sm cursor-pointer hover:text-white/80 transition">
              <input type="checkbox" v-model="useNumbers" class="accent-indigo-500 w-4 h-4" />
              Numbers
            </label>
            <label class="flex items-center gap-3 text-white/50 text-sm cursor-pointer hover:text-white/80 transition">
              <input type="checkbox" v-model="useSymbols" class="accent-indigo-500 w-4 h-4" />
              Symbols
            </label>
          </div>

          <button
            @click="generatePassword"
            class="w-full bg-indigo-600 hover:bg-indigo-500 active:bg-indigo-700 text-white font-bold py-4 rounded-xl text-sm uppercase tracking-widest transition-all duration-200"
          >
            Generate Password
          </button>
        </div>

        <!-- Rechte Seite: Erklärung -->
        <div class="p-10 flex flex-col gap-7">
          <div>
            <h2 class="text-white font-bold text-lg mb-2">Wie sicher ist dein Passwort?</h2>
            <p class="text-white/30 text-xs leading-relaxed">
              Die Stärke basiert auf der
              <span class="text-white/55 font-semibold">Entropie</span>
              — sie misst, wie viele mögliche Kombinationen existieren.
            </p>
          </div>

          <div class="h-px bg-white/[0.06]"></div>

          <!-- Live-Berechnung -->
          <div class="flex flex-col gap-4">
            <div class="flex justify-between items-center">
              <span class="text-white/30 text-xs uppercase tracking-widest">Formel</span>
              <span class="font-mono text-indigo-300/80 text-xs">Länge × log₂(Pool)</span>
            </div>
            <div class="flex justify-between items-center">
              <span class="text-white/30 text-xs uppercase tracking-widest">Deine Werte</span>
              <span class="font-mono text-xs text-white/40">
                {{ passwordLength }} × log₂({{ poolSize }}) =
                <strong class="font-semibold ml-1" :style="{ color: strengthColor }">
                  {{ entropyBits.toFixed(1) }} Bits
                </strong>
              </span>
            </div>
            <div class="flex justify-between items-center">
              <span class="text-white/30 text-xs uppercase tracking-widest">Zeichenpool</span>
              <span class="font-mono text-white/20 text-xs">{{ poolSize }} Zeichen / Stelle</span>
            </div>
          </div>

          <div class="h-px bg-white/[0.06]"></div>

          <div class="flex justify-between items-start">
            <span class="text-white/30 text-xs uppercase tracking-widest mt-0.5">Cracking-Zeit</span>
            <div class="text-right">
              <div class="font-semibold text-base transition-colors" :style="{ color: strengthColor }">
                {{ estimatedCrackTime }}
              </div>
              <div class="text-white/20 text-xs mt-0.5">bei 1 Mrd. Versuche / Sek.</div>
            </div>
          </div>

          <div class="h-px bg-white/[0.06]"></div>

          <!-- Richtwerte -->
          <div class="flex flex-col gap-3">
            <span class="text-white/30 text-xs uppercase tracking-widest">Richtwerte</span>
            <div
              v-for="strengthLevel in [
                { range: '< 40 Bits', label: 'Schwach', color: '#f87171' },
                { range: '40–60 Bits', label: 'Okay', color: '#fbbf24' },
                { range: '60–80 Bits', label: 'Stark', color: '#60a5fa' },
                { range: '> 80 Bits', label: 'Sehr Stark', color: '#34d399' },
              ]"
              :key="strengthLevel.label"
              class="flex items-center gap-2.5"
            >
              <span class="w-1.5 h-1.5 rounded-full flex-shrink-0" :style="{ background: strengthLevel.color }"></span>
              <span class="text-white/30 text-xs">{{ strengthLevel.range }}</span>
              <span class="ml-auto text-xs font-semibold" :style="{ color: strengthLevel.color }">
                {{ strengthLevel.label }}
              </span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
