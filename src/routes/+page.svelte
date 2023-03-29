<script lang="ts">

    type AppData = {
        leftApproachName: string,
        rightApproachName: string,
        timePeriods: TimeLog[]
    };

    type TimeLog = {
        timeStart: Date;
        leftApproach: ApproachData;
        rightApproach: ApproachData;
    }

    type ApproachData = {
        left: number;
        straight: number;
        right: number;
        heavy: number;
        pedestrian: number;
    }

    import Counter from "./Counter.svelte";
    import { persisted } from 'svelte-local-storage-store';
    import * as devalue from 'devalue';
    import Download from "./Download.svelte";
    import { version } from "$app/environment";
    export const appData = persisted<AppData>('trafficLog', {
        leftApproachName: 'Left',
        rightApproachName: 'Right',
        timePeriods: []
    }, {
        serializer: devalue
    });

    function findOrCreateTimePeriodIndex(): number {
        const thisPeriodStart = Math.floor(new Date().getTime()/(1000*60*15))*15*60*1000;
        if (thisPeriodStart > ($appData.timePeriods.at(-1)?.timeStart.getTime() ?? 0)) {
            // New Period
            $appData.timePeriods.push({
                timeStart: new Date(thisPeriodStart),
                leftApproach: {
                    left: 0,
                    straight: 0,
                    right: 0,
                    heavy: 0,
                    pedestrian: 0
                },
                rightApproach: {
                    left: 0,
                    straight: 0,
                    right: 0,
                    heavy: 0,
                    pedestrian: 0
                }
            });
        }

        return $appData.timePeriods.length - 1;
    }

    let time = new Date();


    setInterval(()=>{
        time = new Date();
    }, 100);

    function recordVehicle(approach: 'l' | 'r', description: 'left' | 'straight' | 'right' | 'heavy' | 'pedestrian', diff: number = 1) {
        const timePeriodIndex = findOrCreateTimePeriodIndex();
        $appData.timePeriods[timePeriodIndex][approach === 'l' ? 'leftApproach' : 'rightApproach'][description]++;
    }

    let confirmReset = false;

    function handleResetPress() {
        if (confirmReset) {
            appData.set({
                leftApproachName: 'Left',
                rightApproachName: 'Right',
                timePeriods: []
            });
        }
        confirmReset = !confirmReset;
    }

    function generateCSV() {
        const headerRows = [
            [
                'Time Period',
                `Approach: ${$appData.leftApproachName}`,
                '',
                '',
                '',
                '',
                `Approach: ${$appData.rightApproachName}`,
                '',
                '',
                '',
                ''
            ],
            [
                'Start Time',
                'Left',
                'Straight',
                'Right',
                'Heavy',
                'Pedestrian',
                'Left',
                'Straight',
                'Right',
                'Heavy',
                'Pedestrian'
            ]
        ];
        const dataRows = $appData.timePeriods.map(({timeStart, leftApproach, rightApproach}) => [
            `${timeStart.getHours()}:${timeStart.getMinutes().toString().padStart(2, '0')}`,
            ...[leftApproach,rightApproach].flatMap(({left, straight, right, heavy, pedestrian}) => [left, straight, right, heavy, pedestrian]).map(num => num.toString())
        ]);
        const rows = [
            ...headerRows,
            ...dataRows
        ];
        const csvText = rows.map(row => row.join(',')).join('\n');
        downloadURL = window.URL.createObjectURL(new Blob([csvText], { type: 'text/csv'}));
    }

    let downloadURL = '';

    function handleDownloadCompleted() {
        window.URL.revokeObjectURL(downloadURL);
        downloadURL = '';
    }

</script>

<div class="only screen">
    <header>
        {time?.toLocaleTimeString()}
        <div>
            <button on:click={()=>window.print()}>Print/Save as PDF</button>
            <button on:click={handleResetPress}>
                {#if confirmReset}
                    Are your sure you want to reset?
                {:else}
                    Reset
                {/if}
            </button>
            <button on:click={generateCSV}>Download CSV</button>
        </div>
    </header>
    
    <br>
    
    <div class="display">
        <div class="graphic"></div>
        <div class="counters">        
            <Counter approach="l" {recordVehicle} />
            <div class="modifiers">
                <kbd>+Shift</kbd>
                <span>Heavy</span>
            </div>
            <Counter approach="r" {recordVehicle} />
        </div>
    </div>
    
    <br>
</div>

<table>
    <thead>
        <tr>
            <th>Interval</th>
            <th colspan="5">Approach: <input type="text" bind:value={$appData.leftApproachName}></th>
            <th colspan="5">Approach: <input type="text" bind:value={$appData.rightApproachName}></th>
        </tr>
        <tr>
            <th>Time</th>
            <th>Left</th>
            <th>Straight</th>
            <th>Right</th>
            <th>Heavy</th>
            <th>Pedestrian</th>
            <th>Left</th>
            <th>Straight</th>
            <th>Right</th>
            <th>Heavy</th>
            <th>Pedestrian</th>
        </tr>
    </thead>
    <tbody>
        {#each $appData?.timePeriods as timePeriod (timePeriod.timeStart)}
            <tr>
                <td>{timePeriod.timeStart.getHours()}:{timePeriod.timeStart.getMinutes().toString().padStart(2, '0')}</td>
                <td>{timePeriod.leftApproach.left}</td>
                <td>{timePeriod.leftApproach.straight}</td>
                <td>{timePeriod.leftApproach.right}</td>
                <td>{timePeriod.leftApproach.heavy}</td>
                <td>{timePeriod.leftApproach.pedestrian}</td>
                <td>{timePeriod.rightApproach.left}</td>
                <td>{timePeriod.rightApproach.straight}</td>
                <td>{timePeriod.rightApproach.right}</td>
                <td>{timePeriod.rightApproach.heavy}</td>
                <td>{timePeriod.rightApproach.pedestrian}</td>
            </tr>
        {/each}
    </tbody>
</table>

<br>
<hr>

<details class="only screen">
    <summary>How to Use</summary>
    <p>
        Set the name of the approach roads then use the displayed keyboard shortcuts to count vehicles.<br>
        <kbd>A</kbd>, <kbd>W</kbd>, <kbd>D</kbd> count vehicles turning from the left approach and the <kbd>←</kbd>, <kbd>↑</kbd>, <kbd>→</kbd> arrow keys count vehicles from the right approach.<br>
        Hold <kbd>Shift</kbd> and press a direction key to count a heavy vehicle. Press <kbd>S</kbd> or <kbd>↓</kbd> to count a pedestrian or other vulnerable user crossing the respective approach.
    </p>
    <p>
        You may print the table or save as a PDF using either the normal keyboard shortcut (<kbd>Ctrl</kbd>/<kbd>Command</kbd>+<kbd>P</kbd>) or the "Print/Save as PDF" button at the top-right.<br>
        You may download the raw data as a CSV (open in Excel) using the button at the top-right.<br>
        You can reset all data by using the "Reset" button. You must then confirm that you want to (it's irreversable!) and it will completely reset the app.
    </p>
</details>

<p>
    <span class="only print">Generated By</span> Traffic Counter by Sam Ollari <span class="only screen">(version {version})</span><br>
    <a href="https://github.com/i-am-gizm0/traffic-count">https://github.com/i-am-gizm0/traffic-count</a>
</p>

{#if downloadURL}
    <Download url={downloadURL} name="intersection.csv" on:downloaded={handleDownloadCompleted} />
{/if}

<style>
    header {
        display: flex;
        justify-content: space-between;
        align-items: baseline;
    }

    .counters {
        display: flex;
        justify-content: space-evenly;
    }

    table {
        border-collapse: collapse;
        width: 100%;
    }

    th, td {
        border: 1px solid #000;
        padding: 1ch;
    }

    .modifiers {
        display: grid;
        grid-template-columns: auto auto;
        grid-template-rows: 1fr 1fr;
        column-gap: 2ch;
        justify-items: start;
        align-items: baseline;
    }

    :global(kbd) {
        background-color: #ddd;
        padding: 0.5ch;
        border-radius: 0.5ch;
        text-align: center;
        margin: 0.5ch;
    }

    .only {
        display: none;
    }

    @media only screen {
        .only.screen {
            display: initial;
        }
    }

    @media only screen and (prefers-color-scheme: dark) {
        th, td {
            border-color: #ddd;
        }

        :global(kbd) {
            background-color: #444;
        }
    }

    @media only print {
        th, td {
            text-align: left;
        }

        table {
            border: 1px solid black !important;
        }


        input {
            border: none;
        }

        .only.print {
            display: initial;
        }
    }

    :global(:root) {
        font-family: -apple-system, BlinkMacSystemFont, avenir next, avenir, segoe ui, helvetica neue, helvetica, Cantarell, Ubuntu, roboto, noto, arial, sans-serif;
        color-scheme: light dark;
    }

    p {
        text-align: center;
    }
</style>

<svelte:head>
    <title>Traffic Counter</title>
</svelte:head>
