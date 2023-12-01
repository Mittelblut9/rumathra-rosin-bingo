<template>
    <section class="section-bingo">
        <div class="section-bingo--animation" id="bingo-container">
            <img src="@/assets/gif/winning.gif" alt="bingo" style="visibility: hidden;"/>
            <h1>BINGO!!</h1>
            <table>
                <tr class="user-select-none">
                    <th data-fixed="true">B</th>
                    <th data-fixed="true">I</th>
                    <th data-fixed="true">N</th>
                    <th data-fixed="true">G</th>
                    <th data-fixed="true">O</th>
                </tr>
                <tr v-for="tableData in tableData" :key="tableData" id="bingo-content">
                    <td v-for="tableContent in tableData" :key="tableContent" @click="selectTarget" @contextmenu="rightClickHandler($event)">
                        <BingoBoxAtom 
                            :selected="false" 
                            :text="tableContent.text" 
                            :multiSelectNumber="tableContent.multiSelectNumber"
                            :multiSelectOptions="JSON.stringify(tableContent?.multiSelectOptions)"
                        />
                    </td>
                </tr>
            </table>
        </div>
    </section>
</template>

<script>
import tableData from '@/assets/json/bingo.json';
import BingoBoxAtom from '@/atoms/bingo-box.atom.vue';
export default {
    name: 'BingoComponent',
    data() {
        return {
            tableData: tableData.bingo,
            bingoTriggered: false
        };
    },
    components: {
        BingoBoxAtom
    },
    methods: {
        rightClickHandler(e) {
            e.preventDefault();

            const target = e.target;
            const input = target.firstElementChild;

            if(input.dataset.fixed === 'true') {
                return;
            }

            input.dataset.selected = false;
            input.dataset.timesPressed = 0;
            input.dataset.timesRounds = 0;
            target.style.backgroundColor = '';

            const isMultiSelect = input.dataset.multiSelectNumber > 0;
            if(!isMultiSelect) {
                return;
            }

            target.querySelectorAll('.bingo-box-multi--item').forEach(div => {
                div.style.backgroundColor = '';
            });
        },
        selectTarget(e) {
            const target = e.target;
            const input = target.querySelector('input');
            let targetDiv;

            const fixed = input.dataset.fixed === 'true';
            if(fixed) {
                return;
            }
            
            let timesPressed = parseInt(input.dataset.timesPressed);
            let timesRounds = parseInt(input.dataset.timesRounds);

            let selected = input.dataset.selected === 'true';
            let color;
            let resetColor = false;
            let resetTimesPressed = false;

            const multiSelectNumber = parseInt(input.dataset.multiSelectNumber);
            const isMultiSelect = multiSelectNumber > 0;
            const multiSelectOptions = tableData.options;
            
            if (isMultiSelect && timesPressed > 0)  {
                color = multiSelectOptions[`color${timesRounds}`];
            }else {
                color = multiSelectOptions['color0'];
                selected = !selected;
            }

            if(isMultiSelect) {
                selected = false;
                targetDiv = target.querySelector(`.bingo-box-multi--item input[data-multi-select-number="${timesPressed + 1}"]`);
                if(!targetDiv) {
                    targetDiv = target.querySelector('.bingo-box-multi--item input[data-multi-select-number="1"]');
                    color = multiSelectOptions[`color${timesRounds + 1}`];
                    input.dataset.timesRounds = timesRounds + 1;
                    timesPressed = 0;
                }

                const nextDivExists = target.querySelector(`.bingo-box-multi--item input[data-multi-select-number="${timesPressed + 2}"]`);
                if(!nextDivExists) {
                    selected = true;
                }

                targetDiv = targetDiv.parentElement;
            }else {
                targetDiv = target;
            }

            targetDiv.style.backgroundColor = !selected && !isMultiSelect ? '' : color;
            input.dataset.selected = selected;
            input.dataset.timesPressed = resetTimesPressed ? 0 : timesPressed + 1;


            if(resetColor) {
                targetDiv.style.backgroundColor = '';
            }

            this.checkIfBingo((isBingo) => {
                if(isBingo) {
                    if(this.bingoTriggered) {
                        return;
                    }
                    this.bingoTriggered = true;
                    this.animateBingo();
                }
            });
        },

        checkIfBingo(cb) {
            const table = document.querySelector('.section-bingo table');
            const tableRows = table.querySelectorAll('#bingo-content');
            const tableColumns = table.querySelectorAll('td');

            let isBingo = false;

            tableRows.forEach(row => {
                if(isBingo) {
                    return;
                }
                const rowCells = row.querySelectorAll('td');
                const rowCellsLength = rowCells.length;

                let rowCellsSelected = 0;
                rowCells.forEach(cell => {
                    const input = cell.querySelector('input');
                    const selected = input.dataset.selected === 'true';
                    if(selected) {
                        rowCellsSelected++;
                    }
                });

                isBingo = rowCellsSelected === rowCellsLength;

                cb(isBingo);
            });

            tableColumns.forEach(column => {
                if(isBingo) {
                    return;
                }
                const columnCells = table.querySelectorAll(`td:nth-child(${column.cellIndex + 1})`);
                const columnCellsLength = columnCells.length;

                let columnCellsSelected = 0;
                columnCells.forEach(cell => {
                    const input = cell.querySelector('input');
                    const selected = input.dataset.selected === 'true';
                    if(selected) {
                        columnCellsSelected++;
                    }
                });

                isBingo = columnCellsSelected === columnCellsLength;
                cb(isBingo);
            });

            const diagonale = () => {
                const firstCell = table.querySelector('#bingo-content:nth-child(2) td:first-child');
                const secondCell = table.querySelector('#bingo-content:nth-child(3) td:nth-child(2)');
                const thirdCell = table.querySelector('#bingo-content:nth-child(4) td:nth-child(3)');
                const fourthCell = table.querySelector('#bingo-content:nth-child(5) td:nth-child(4)');
                const fifthCell = table.querySelector('#bingo-content:nth-child(6) td:nth-child(5)');

                const firstCellSelected = firstCell.querySelector('input').dataset.selected === 'true';
                const secondCellSelected = secondCell.querySelector('input').dataset.selected === 'true';
                const thirdCellSelected = thirdCell.querySelector('input').dataset.selected === 'true';
                const fourthCellSelected = fourthCell.querySelector('input').dataset.selected === 'true';
                const fifthCellSelected = fifthCell.querySelector('input').dataset.selected === 'true';

                return firstCellSelected && secondCellSelected && thirdCellSelected && fourthCellSelected && fifthCellSelected;
            };

            const diagonale2 = () => {
                const firstCell = table.querySelector('#bingo-content:nth-child(2) td:nth-child(5)');
                const secondCell = table.querySelector('#bingo-content:nth-child(3) td:nth-child(4)');
                const thirdCell = table.querySelector('#bingo-content:nth-child(4) td:nth-child(3)');
                const fourthCell = table.querySelector('#bingo-content:nth-child(5) td:nth-child(2)');
                const fifthCell = table.querySelector('#bingo-content:nth-child(6) td:nth-child(1)');

                const firstCellSelected = firstCell.querySelector('input').dataset.selected === 'true';
                const secondCellSelected = secondCell.querySelector('input').dataset.selected === 'true';
                const thirdCellSelected = thirdCell.querySelector('input').dataset.selected === 'true';
                const fourthCellSelected = fourthCell.querySelector('input').dataset.selected === 'true';
                const fifthCellSelected = fifthCell.querySelector('input').dataset.selected === 'true';

                return firstCellSelected && secondCellSelected && thirdCellSelected && fourthCellSelected && fifthCellSelected;
            };

            cb(diagonale());
            cb(diagonale2());
        },

        animateBingo() {
            const bingoContainer = document.querySelector('.section-bingo--animation');
            bingoContainer.classList.add('section-bingo--animation--active');

            bingoContainer.querySelector('h1').style.opacity = 1;

            setTimeout(() => {
                bingoContainer.querySelector('h1').style.opacity = 0;
                bingoContainer.classList.remove('section-bingo--animation--active');
            }, 5000);
        }
    }
};
</script>