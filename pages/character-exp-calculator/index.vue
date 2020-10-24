<template>
    <div>
        <b-row>
            <b-col lg="6" sm="12">
                <b-form-group
                    label="World Level"
                    label-size="lg"
                    label-class="font-weight-bold pt-0"
                >
                    <b-form-select
                        id="world-lvl"
                        v-model="worldLvl"
                        :options="worldLvlOptions"
                    />
                </b-form-group>
                <b-form-group
                    label="Character"
                    label-size="lg"
                    label-class="font-weight-bold pt-0"
                >
                    <b-row>
                        <b-col lg="6" md="12">
                            <b-form-group label="Character Level">
                                <b-form-select
                                    id="character-lvl"
                                    v-model="character"
                                    :options="lvlOptions"
                                />
                            </b-form-group>
                        </b-col>
                        <b-col lg="6" md="12">
                            <b-form-group label="Target Character Level">
                                <b-form-select
                                    id="target-character-lvl"
                                    v-model="targetCharacter"
                                    :options="lvlOptions"
                                />
                            </b-form-group>
                        </b-col>
                    </b-row>
                    <b-form-group label="Current EXP">
                        <b-form-input
                            id="cur-exp"
                            v-model.number="currentExp"
                            type="number"
                            min="0"
                            :max="character.expToNextLvl - 1"
                            step="1"
                            pattern="[0-9]"
                            @input="validateCurrentExp"
                        />
                    </b-form-group>
                </b-form-group>
                <b-form-group
                    label="EXP Books"
                    label-size="lg"
                    label-class="font-weight-bold pt-0"
                >
                    <b-form-group label="Hero's Wit (20,000 EXP)">
                        <b-form-input
                            id="purple-books"
                            v-model.number="expBooks.purple"
                            type="number"
                            min="0"
                            step="1"
                            pattern="[0-9]"
                        />
                    </b-form-group>
                    <b-form-group label="Adventurer's Experience (5000 EXP)">
                        <b-form-input
                            id="red-books"
                            v-model.number="expBooks.red"
                            type="number"
                            min="0"
                            step="1"
                            pattern="[0-9]"
                        />
                    </b-form-group>
                    <b-form-group label="Adventurer's Advice (1000 EXP)">
                        <b-form-input
                            id="white-books"
                            v-model.number="expBooks.white"
                            type="number"
                            min="0"
                            step="1"
                            pattern="[0-9]"
                        />
                    </b-form-group>
                </b-form-group>
                <b-form-group
                    label="Mora"
                    label-size="lg"
                    label-class="font-weight-bold pt-0"
                >
                    <b-form-input
                        id="mora"
                        v-model.number="mora"
                        type="number"
                        min="0"
                        step="1"
                        pattern="[0-9]"
                    />
                </b-form-group>
            </b-col>
            <b-col lg="6" sm="12">
                <h3>Results:</h3>
                <h4>Summary</h4>
                <b-table
                    bordered
                    fixed
                    :items="summaryTableData"
                    thead-class="hidden-header"
                />
                <h4>EXP</h4>
                <b-table
                    bordered
                    fixed
                    :items="expTableData"
                    thead-class="hidden-header"
                />
                <h4>Mora</h4>
                <b-table
                    bordered
                    fixed
                    :items="moraTableData"
                    thead-class="hidden-header"
                />
            </b-col>
        </b-row>
    </div>
</template>

<script>
const MORA_PER_EXP = 0.2;
const PURPLE_BOOK_EXP = 20000;
const RED_BOOK_EXP = 5000;
const WHITE_BOOK_EXP = 1000;
const RESIN_PER_RUN = 20;

const getLvlOptions = () =>
    import("@/static/json/character-level-options.json").then(
        (m) => m.default || m
    );
const getWorldLvlOptions = () =>
    import("@/static/json/world-level-options.json").then(
        (m) => m.default || m
    );

export default {
    async asyncData({ params }) {
        const lvlOptions = await getLvlOptions();
        const worldLvlOptions = await getWorldLvlOptions();
        return { lvlOptions, worldLvlOptions };
    },
    data() {
        return {
            worldLvl: {
                lvl: 0,
                expBooks: {
                    purple: 0,
                    red: 3,
                    white: 7,
                },
                mora: 12000,
            },
            currentExp: 0,
            character: {
                lvl: 1,
                baseTotalExp: 0,
                expToNextLvl: 1000,
                ascension_cost: 0,
            },
            targetCharacter: {
                lvl: 2,
                baseTotalExp: 1000,
                expToNextLvl: 1325,
                ascension_cost: 0,
            },
            expBooks: {
                white: 0,
                red: 0,
                purple: 0,
            },
            mora: 0,
        };
    },
    computed: {
        totalExp() {
            return this.character.baseTotalExp + this.currentExp;
        },
        expToTargetLvl() {
            return this.targetCharacter.baseTotalExp - this.totalExp;
        },
        totalAscensionCost() {
            let total = 0;
            const curLvlIndex = this.character.lvl - 1;
            const targetLvlIndex = this.targetCharacter.lvl - 1;

            for (let i = curLvlIndex + 1; i <= targetLvlIndex; i++) {
                total += this.lvlOptions[i].value.ascension_cost;
            }

            return total;
        },
        moraToTargetLvl() {
            const lvlingCost = this.expToTargetLvl * MORA_PER_EXP;

            return lvlingCost + this.totalAscensionCost;
        },
        expPerLeylineRun() {
            return this.getExpFromBooks(this.worldLvl.expBooks);
        },
        expAvailable() {
            return this.getExpFromBooks(this.expBooks);
        },
        blueLeylineRuns() {
            const remainingExp = this.expToTargetLvl - this.expAvailable;
            let runs = Math.ceil(remainingExp / this.expPerLeylineRun);
            runs = runs > 0 ? runs : 0;

            const ret = {
                runs,
                cost: runs * RESIN_PER_RUN,
            };
            return ret;
        },
        goldLeyLineRuns() {
            const remainingMora = this.moraToTargetLvl - this.mora;
            let runs = Math.ceil(remainingMora / this.worldLvl.mora);
            runs = runs > 0 ? runs : 0;

            const ret = {
                runs,
                cost: runs * RESIN_PER_RUN,
            };
            return ret;
        },
        summaryTableData() {
            return [
                {
                    key: "Total EXP to target lvl",
                    data: this.expToTargetLvl,
                    _cellVariants: { key: "dark" },
                },
                {
                    key: "Total Mora to target lvl",
                    data: this.moraToTargetLvl,
                    _cellVariants: { key: "dark" },
                },
                {
                    key: "Blue Leyline runs",
                    data: this.blueLeylineRuns.runs,
                    _cellVariants: { key: "dark" },
                },
                {
                    key: "Blue Leyline resin cost",
                    data: this.blueLeylineRuns.cost,
                    _cellVariants: { key: "dark" },
                },
                {
                    key: "Gold Leyline runs",
                    data: this.goldLeyLineRuns.runs,
                    _cellVariants: { key: "dark" },
                },
                {
                    key: "Gold Leyline resin cost",
                    data: this.goldLeyLineRuns.cost,
                    _cellVariants: { key: "dark" },
                },
                {
                    key: "Total resin cost",
                    data: this.blueLeylineRuns.cost + this.goldLeyLineRuns.cost,
                    _cellVariants: { key: "dark" },
                },
            ];
        },
        expTableData() {
            const expBooksPerRun = `
                Purple: ${this.worldLvl.expBooks.purple},
                Red: ${this.worldLvl.expBooks.red},
                White: ${this.worldLvl.expBooks.white}
            `;

            return [
                {
                    key: "Total EXP to target lvl",
                    data: this.expToTargetLvl,
                    _cellVariants: { key: "dark" },
                },
                {
                    key: "Total EXP per Blue Leyline run",
                    data: this.expPerLeylineRun,
                    _cellVariants: { key: "dark" },
                },
                {
                    key: "Books per Leyline run",
                    data: expBooksPerRun,
                    _cellVariants: { key: "dark" },
                },
                {
                    key: "Leyline runs",
                    data: this.blueLeylineRuns.runs,
                    _cellVariants: { key: "dark" },
                },
                {
                    key: "Resin cost",
                    data: this.blueLeylineRuns.cost,
                    _cellVariants: { key: "dark" },
                },
            ];
        },
        moraTableData() {
            return [
                {
                    key: "Total Mora to target lvl",
                    data: this.moraToTargetLvl,
                    _cellVariants: { key: "dark" },
                },
                {
                    key: "Mora per Gold Leyline run",
                    data: this.worldLvl.mora,
                    _cellVariants: { key: "dark" },
                },
                {
                    key: "Leyline runs",
                    data: this.goldLeyLineRuns.runs,
                    _cellVariants: { key: "dark" },
                },
                {
                    key: "Resin cost",
                    data: this.goldLeyLineRuns.cost,
                    _cellVariants: { key: "dark" },
                },
            ];
        },
    },
    methods: {
        validateCurrentExp() {
            if (isNaN(this.currentExp)) {
                this.currentExp = 0;
            } else if (this.currentExp >= this.character.expToNextLvl) {
                this.currentExp = this.character.expToNextLvl - 1;
            }
        },
        getExpFromBooks(expBooks) {
            const purpleBookExp = expBooks.purple * PURPLE_BOOK_EXP;
            const redBookExp = expBooks.red * RED_BOOK_EXP;
            const whiteBookExp = expBooks.white * WHITE_BOOK_EXP;

            return purpleBookExp + redBookExp + whiteBookExp;
        },
    },
};
</script>

<style lang="scss" scoped></style>

<style>
.hidden-header {
    display: none;
}
</style>
