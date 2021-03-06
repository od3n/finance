<template>
    <div>
        <button @click="openModal" class="p-2 ml-4 focus:outline-none rounded-lg flex items-center hover:shadow" v-dark-mode-button>
            <zondicon icon="add-outline" class="w-6 h-6 fill-current" />
            <span class="ml-2 font-medium">New Group</span>
        </button>

        <modal ref="groupingModal" @closed="closeModal">
            <div class="flex flex-wrap">
                <div class="w-full">
                    <label class="block uppercase tracking-wide text-xs font-bold mb-2" v-dark-mode-dark-text>
                        Group Name
                    </label>
                    <input v-model="form.name" class="appearance-none block w-full  rounded py-3 px-4 leading-tight focus:outline-none" type="text" placeholder="video games" v-dark-mode-input/>
                </div>

                <div class="w-full py-4 ">
                    <div class="block uppercase tracking-wide text-xs font-bold mb-2" v-dark-mode-dark-text>
                        Conditions
                        <div class="text-xs font-normal normal-case" v-dark-mode-light-gray-text>
                            Adding conditions to groups allows our system to automatically apply these groups to your transactions.
                            Not adding any conditions will automatically apply the group to all transactions.
                        </div>
                    </div>

                    <div class="flex w-full mt-4" v-for="condition in form.conditions">
                        <div class="flex-1">
                            <div class="block uppercase tracking-wide text-xs mb-2 font-semibold"  v-dark-mode-dark-text>
                                Parameter
                                <div class="text-xs font-normal tracking-tight" v-dark-mode-light-text>
                                    The field who's value we will compare against
                                </div>
                            </div>
                            <select v-model="condition.parameter" class="appearance-none block w-full  rounded py-1 px-2 leading-tight focus:outline-none"  v-dark-mode-input>
                                <option v-for="parameter in parameters" :value="parameter.value">{{ parameter.name }}</option>
                            </select>
                        </div>
                        <div class="flex-1 ml-4">
                            <div class="block uppercase tracking-wide text-xs mb-2 font-semibold" v-dark-mode-dark-text>
                                Comparator
                                <div class="text-xs font-normal tracking-tight" v-dark-mode-light-text>
                                    How we compare the parameter to the value
                                </div>
                            </div>

                            <select v-model="condition.comparator" class="appearance-none block w-full  rounded py-1 px-2 leading-tight focus:outline-none"  v-dark-mode-input>
                                <option v-for="comparator in comparators" :value="comparator.value">{{ comparator.name }}</option>
                            </select>
                        </div>
                        <div class="flex-1 ml-4">
                            <div class="block uppercase tracking-wide text-xs mb-2 font-semibold" v-dark-mode-dark-text>
                                Value
                                <div class="text-xs font-normal tracking-tight" v-dark-mode-light-text>
                                    What we are comparing the transaction to
                                </div>
                            </div>
                            <input v-model="condition.value" class="appearance-none block w-full  rounded py-1 px-2 leading-tight focus:outline-none" type="text" placeholder="STEAMGAMES.COM"  v-dark-mode-input/>
                        </div>

                        <div class="w-6 ml-2 flex justify-center items-end mb-1">
                            <button class="text-red-600 h-6" @click.prevent="() => deleteCondition(condition)">
                                <svg class="w-6" fill="currentColor" viewBox="0 0 20 20"><path fill-rule="evenodd" d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z" clip-rule="evenodd"></path></svg>
                            </button>
                        </div>
                    </div>
                    <div class="w-full text-xs" v-dark-mode-light-text>
                        <label class="mt-2 flex w-full items-center">
                            <input type="checkbox" v-model="form.must_all_conditions_pass" />
                            <span class="ml-2">Must all conditions be true?</span>
                        </label>
                    </div>
                    <button @click="addCondition" class="mt-4 px-2 py-1 text-sm focus:outline-none rounded-lg flex items-center hover:shadow" v-dark-mode-button>
                        <zondicon icon="add-outline" class="fill-current w-4 h-4" />
                        <span class="ml-2">Add condition</span>
                    </button>
                </div>

                <div class="w-full py-4">
                    <button @click.prevent="saveGroup" class="py-2 px-4 border-transparent focus:outline-none rounded-lg flex items-center hover:shadow" v-dark-mode-button>
                        <zondicon v-if="saving" icon="refresh" class="w-4 h-4 rotate" />
                        Sav<span v-if="!saving">e</span><span v-else>ing</span>
                    </button>
                </div>
            </div>
        </modal>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                form: {
                    name: '',
                    conditions: [],
                    must_all_conditions_pass: true
                },
                saving: false,
                parameters: require('../condition-parameters'),
                comparators: require('../condition-comparator'),
            }
        },
        methods: {
            closeModal() {
                this.$refs.groupingModal.hide();
            },
            openModal() {
                this.$refs.groupingModal.show();
            },
            addCondition() {
                this.form.conditions.push({
                    value: '',
                    comparator: 'LIKE',
                    parameter: 'name',
                })
            },
            async saveGroup() {
                this.saving = true;

                await this.$store.dispatch('saveGroup', this.form);

                this.saving = false;

                setTimeout(() => this.closeModal(), 300);
            },

            async deleteCondition(condition) {
                if (!condition.id) {
                    this.form = {
                        ...this.form,
                        conditionals: this.form.conditionals.filter(con => !(
                            con.value === condition.value && con.parameter === condition.parameter && con.comparator === condition.comparator
                        ))
                    }
                    return;
                }
                await this.$store.dispatch('deleteGroupCondition', {
                    tag: this.tag,
                    condition
                });

                setTimeout(() => this.closeModal(), 300);
                await this.$store.dispatch('fetchGroups');
            },
        }
    }
</script>

<style scoped>

</style>
