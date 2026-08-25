/* =========================================================
   GYROLAB - GYROSCOPIC COUPLE ANALYZER
   Main JavaScript
========================================================= */


/* =========================================================
   GLOBAL VARIABLES
========================================================= */

let analysisChart = null;

let currentGraph = "speed";

let animationAngle = 0;

let inputMode = "standard";


/* =========================================================
   STANDARD PRESETS
========================================================= */

const presets = {

    standard1: {
        mass: 5,
        radius: 0.20,
        wheelSpeed: 1000,
        precessionSpeed: 60
    },

    standard2: {
        mass: 10,
        radius: 0.25,
        wheelSpeed: 1200,
        precessionSpeed: 75
    },

    standard3: {
        mass: 8,
        radius: 0.18,
        wheelSpeed: 2000,
        precessionSpeed: 100
    }

};


/* =========================================================
   PAGE INITIALIZATION
========================================================= */

document.addEventListener("DOMContentLoaded", function () {

    applyPreset();

    calculate();

    initializeChart();

    drawGyroscope();

    startAnimation();

});


/* =========================================================
   HELPER
========================================================= */

function getNumber(id) {

    const element = document.getElementById(id);

    if (!element) {
        return 0;
    }

    const value = parseFloat(element.value);

    if (isNaN(value)) {
        return 0;
    }

    return value;
}


/* =========================================================
   GET CURRENT INPUT VALUES
========================================================= */

function getInputs() {

    return {

        mass: getNumber("mass"),

        radius: getNumber("radius"),

        wheelSpeed: getNumber("wheelSpeed"),

        precessionSpeed: getNumber("precessionSpeed"),

        wheelType:
            document.getElementById("wheelType").value

    };

}


/* =========================================================
   CALCULATE GYROSCOPIC COUPLE
========================================================= */

function calculate() {

    const data = getInputs();


    /* -----------------------------------------------------
       Basic validation
    ----------------------------------------------------- */

    if (
        data.mass <= 0 ||
        data.radius <= 0 ||
        data.wheelSpeed <= 0 ||
        data.precessionSpeed <= 0
    ) {

        return;

    }


    /* -----------------------------------------------------
       Moment of inertia
    ----------------------------------------------------- */

    let inertia;

    if (data.wheelType === "ring") {

        // Thin ring
        inertia = data.mass * Math.pow(data.radius, 2);

    } else {

        // Solid disc
        inertia =
            0.5 *
            data.mass *
            Math.pow(data.radius, 2);

    }


    /* -----------------------------------------------------
       Radius of gyration
    ----------------------------------------------------- */

    const gyration =
        Math.sqrt(inertia / data.mass);


    /* -----------------------------------------------------
       Convert RPM → rad/s
    ----------------------------------------------------- */

    const omega =
        (2 * Math.PI * data.wheelSpeed) / 60;


    const omegaP =
        (2 * Math.PI * data.precessionSpeed) / 60;


    /* -----------------------------------------------------
       Gyroscopic Couple
    ----------------------------------------------------- */

    const couple =
        inertia *
        omega *
        omegaP;


    /* -----------------------------------------------------
       Update UI
    ----------------------------------------------------- */

    updateValue(
        "couple",
        couple.toFixed(2)
    );


    updateValue(
        "inertia",
        inertia.toFixed(4)
    );


    updateValue(
        "gyration",
        gyration.toFixed(4)
    );


    updateValue(
        "omega",
        omega.toFixed(2)
    );


    updateValue(
        "omegaP",
        omegaP.toFixed(2)
    );


    /* -----------------------------------------------------
       Update step-by-step calculation
    ----------------------------------------------------- */

    updateValue(
        "stepInertia",
        `I = ${inertia.toFixed(4)} kg·m²`
    );


    updateValue(
        "stepOmega",
        `ω = ${omega.toFixed(2)} rad/s`
    );


    updateValue(
        "stepOmegaP",
        `ωₚ = ${omegaP.toFixed(2)} rad/s`
    );


    updateValue(
        "stepCouple",
        `C = ${couple.toFixed(2)} N·m`
    );


    /* -----------------------------------------------------
       Visualization values
    ----------------------------------------------------- */

    updateValue(
        "visualSpeed",
        `${data.wheelSpeed.toFixed(0)} RPM`
    );


    updateValue(
        "visualPrecession",
        `${data.precessionSpeed.toFixed(0)} RPM`
    );


    updateValue(
        "visualCouple",
        `${couple.toFixed(2)} N·m`
    );


    /* -----------------------------------------------------
       Validation
    ----------------------------------------------------- */

    updateValue(
        "validationCalculated",
        couple.toFixed(2)
    );


    calculateValidation();


    /* -----------------------------------------------------
       Update graph
    ----------------------------------------------------- */

    updateAnalysisChart();


    /* -----------------------------------------------------
       Redraw wheel
    ----------------------------------------------------- */

    drawGyroscope();

}


/* =========================================================
   UPDATE HTML VALUE
========================================================= */

function updateValue(id, value) {

    const element = document.getElementById(id);

    if (element) {

        element.textContent = value;

    }

}


/* =========================================================
   SLIDER → NUMBER INPUT
========================================================= */

function syncSlider(parameter) {

    const slider =
        document.getElementById(parameter + "Slider");

    const input =
        document.getElementById(parameter);


    if (!slider || !input) {
        return;
    }


    input.value = slider.value;


    calculate();

}


/* =========================================================
   NUMBER INPUT → SLIDER
========================================================= */

function syncInput(parameter) {

    const input =
        document.getElementById(parameter);

    const slider =
        document.getElementById(parameter + "Slider");


    if (!input || !slider) {
        return;
    }


    slider.value = input.value;


    calculate();

}


/* =========================================================
   APPLY STANDARD PRESET
========================================================= */

function applyPreset() {

    const presetElement =
        document.getElementById("preset");


    if (!presetElement) {
        return;
    }


    const selected =
        presets[presetElement.value];


    if (!selected) {
        return;
    }


    setInput(
        "mass",
        selected.mass
    );


    setInput(
        "radius",
        selected.radius
    );


    setInput(
        "wheelSpeed",
        selected.wheelSpeed
    );


    setInput(
        "precessionSpeed",
        selected.precessionSpeed
    );


    calculate();

}


/* =========================================================
   SET INPUT + SLIDER
========================================================= */

function setInput(parameter, value) {

    const input =
        document.getElementById(parameter);

    const slider =
        document.getElementById(parameter + "Slider");


    if (input) {

        input.value = value;

    }


    if (slider) {

        slider.value = value;

    }

}


/* =========================================================
   INPUT MODE
========================================================= */

function setInputMode(mode) {

    inputMode = mode;


    const standardButton =
        document.getElementById("standardModeBtn");

    const customButton =
        document.getElementById("customModeBtn");

    const presetGroup =
        document.getElementById("presetGroup");


    if (mode === "standard") {

        standardButton.classList.add("active");

        customButton.classList.remove("active");

        presetGroup.style.display = "block";

        applyPreset();

    } else {

        standardButton.classList.remove("active");

        customButton.classList.add("active");

        presetGroup.style.display = "none";

    }

}


/* =========================================================
   RESET
========================================================= */

function resetInputs() {

    document.getElementById("wheelType").value =
        "disc";


    document.getElementById("preset").value =
        "standard1";


    setInput(
        "mass",
        5
    );


    setInput(
        "radius",
        0.20
    );


    setInput(
        "wheelSpeed",
        1000
    );


    setInput(
        "precessionSpeed",
        60
    );


    const experimental =
        document.getElementById("experimentalCouple");


    if (experimental) {

        experimental.value = "";

    }


    calculate();

}


/* =========================================================
   CALCULATION TABS
========================================================= */

function openCalcTab(tabName) {

    const contents =
        document.querySelectorAll(
            ".calc-tab-content"
        );


    contents.forEach(function (content) {

        content.classList.remove("active");

    });


    const tabs =
        document.querySelectorAll(
            ".calc-tab"
        );


    tabs.forEach(function (tab) {

        tab.classList.remove("active");

    });


    const selectedContent =
        document.getElementById(
            tabName + "Tab"
        );


    if (selectedContent) {

        selectedContent.classList.add("active");

    }


    tabs.forEach(function (tab) {

        const text =
            tab.textContent
                .trim()
                .toLowerCase();


        if (
            (tabName === "formula" &&
                text.includes("formula")) ||

            (tabName === "steps" &&
                text.includes("step")) ||

            (tabName === "assumptions" &&
                text.includes("assumption"))
        ) {

            tab.classList.add("active");

        }

    });

}


/* =========================================================
   CHART INITIALIZATION
========================================================= */

function initializeChart() {

    const canvas =
        document.getElementById(
            "analysisChart"
        );


    if (!canvas) {
        return;
    }


    const context =
        canvas.getContext("2d");


    analysisChart =
        new Chart(
            context,
            {

                type: "line",

                data: {

                    labels: [],

                    datasets: [
                        {

                            label:
                                "Gyroscopic Couple (N·m)",

                            data: [],

                            borderWidth: 3,

                            pointRadius: 4,

                            tension: 0.35,

                            fill: false

                        }
                    ]

                },


                options: {

                    responsive: true,

                    maintainAspectRatio: false,

                    interaction: {

                        intersect: false,

                        mode: "index"

                    },


                    plugins: {

                        legend: {

                            display: true

                        }

                    },


                    scales: {

                        x: {

                            title: {

                                display: true,

                                text:
                                    "Parameter"

                            }

                        },


                        y: {

                            beginAtZero: true,

                            title: {

                                display: true,

                                text:
                                    "Gyroscopic Couple (N·m)"

                            }

                        }

                    }

                }

            }
        );


    updateAnalysisChart();

}


/* =========================================================
   CREATE GRAPH DATA
========================================================= */

function generateGraphData(type) {

    const data =
        getInputs();


    let labels = [];

    let values = [];


    const points = 12;


    /* -----------------------------------------------------
       Wheel Speed
    ----------------------------------------------------- */

    if (type === "speed") {

        const min = 200;

        const max = Math.max(
            data.wheelSpeed * 1.8,
            1500
        );


        for (
            let i = 0;
            i < points;
            i++
        ) {

            const speed =
                min +
                ((max - min) * i) /
                (points - 1);


            const omega =
                (2 * Math.PI * speed) / 60;


            const omegaP =
                (2 * Math.PI *
                    data.precessionSpeed) /
                60;


            const inertia =
                getInertia(
                    data.mass,
                    data.radius,
                    data.wheelType
                );


            const couple =
                inertia *
                omega *
                omegaP;


            labels.push(
                Math.round(speed)
            );


            values.push(
                Number(couple.toFixed(2))
            );

        }

    }


    /* -----------------------------------------------------
       Precession Speed
    ----------------------------------------------------- */

    else if (type === "precession") {

        const min = 5;

        const max =
            Math.max(
                data.precessionSpeed * 2,
                150
            );


        for (
            let i = 0;
            i < points;
            i++
        ) {

            const speed =
                min +
                ((max - min) * i) /
                (points - 1);


            const omega =
                (2 * Math.PI *
                    data.wheelSpeed) /
                60;


            const omegaP =
                (2 * Math.PI * speed) /
                60;


            const inertia =
                getInertia(
                    data.mass,
                    data.radius,
                    data.wheelType
                );


            const couple =
                inertia *
                omega *
                omegaP;


            labels.push(
                Math.round(speed)
            );


            values.push(
                Number(couple.toFixed(2))
            );

        }

    }


    /* -----------------------------------------------------
       MASS
    ----------------------------------------------------- */

    else if (type === "mass") {

        const min = 1;

        const max = 20;


        for (
            let i = 0;
            i < points;
            i++
        ) {

            const mass =
                min +
                ((max - min) * i) /
                (points - 1);


            const inertia =
                getInertia(
                    mass,
                    data.radius,
                    data.wheelType
                );


            const omega =
                (2 * Math.PI *
                    data.wheelSpeed) /
                60;


            const omegaP =
                (2 * Math.PI *
                    data.precessionSpeed) /
                60;


            const couple =
                inertia *
                omega *
                omegaP;


            labels.push(
                Number(mass.toFixed(1))
            );


            values.push(
                Number(couple.toFixed(2))
            );

        }

    }


    /* -----------------------------------------------------
       RADIUS
    ----------------------------------------------------- */

    else if (type === "radius") {

        const min = 0.05;

        const max = 0.50;


        for (
            let i = 0;
            i < points;
            i++
        ) {

            const radius =
                min +
                ((max - min) * i) /
                (points - 1);


            const inertia =
                getInertia(
                    data.mass,
                    radius,
                    data.wheelType
                );


            const omega =
                (2 * Math.PI *
                    data.wheelSpeed) /
                60;


            const omegaP =
                (2 * Math.PI *
                    data.precessionSpeed) /
                60;


            const couple =
                inertia *
                omega *
                omegaP;


            labels.push(
                Number(radius.toFixed(2))
            );


            values.push(
                Number(couple.toFixed(2))
            );

        }

    }


    return {
        labels,
        values
    };

}


/* =========================================================
   MOMENT OF INERTIA FUNCTION
========================================================= */

function getInertia(
    mass,
    radius,
    wheelType
) {

    if (wheelType === "ring") {

        return (
            mass *
            Math.pow(radius, 2)
        );

    }


    return (
        0.5 *
        mass *
        Math.pow(radius, 2)
    );

}


/* =========================================================
   UPDATE ANALYSIS GRAPH
========================================================= */

function updateAnalysisChart() {

    if (!analysisChart) {
        return;
    }


    const graph =
        generateGraphData(
            currentGraph
        );


    let xTitle =
        "Wheel Speed (RPM)";


    if (currentGraph === "precession") {

        xTitle =
            "Precession Speed (RPM)";

    }


    if (currentGraph === "mass") {

        xTitle =
            "Wheel Mass (kg)";

    }


    if (currentGraph === "radius") {

        xTitle =
            "Wheel Radius (m)";

    }


    analysisChart.data.labels =
        graph.labels;


    analysisChart.data.datasets[0].data =
        graph.values;


    analysisChart.options.scales.x.title.text =
        xTitle;


    analysisChart.update();

}


/* =========================================================
   CHANGE GRAPH
========================================================= */

function showGraph(type) {

    currentGraph = type;


    const buttons =
        document.querySelectorAll(
            ".analysis-btn"
        );


    buttons.forEach(function (button) {

        button.classList.remove(
            "active"
        );

    });


    const clickedButton =
        Array.from(buttons).find(
            button =>
                button.textContent
                    .toLowerCase()
                    .includes(
                        getGraphButtonText(type)
                    )
        );


    if (clickedButton) {

        clickedButton.classList.add(
            "active"
        );

    }


    updateAnalysisChart();

}


/* =========================================================
   GRAPH BUTTON TEXT
========================================================= */

function getGraphButtonText(type) {

    if (type === "speed") {

        return "wheel";

    }


    if (type === "precession") {

        return "precession";

    }


    if (type === "mass") {

        return "mass";

    }


    if (type === "radius") {

        return "radius";

    }


    return "";

}


/* =========================================================
   VALIDATION
========================================================= */

function calculateValidation() {

    const calculatedElement =
        document.getElementById(
            "validationCalculated"
        );


    const experimentalElement =
        document.getElementById(
            "experimentalCouple"
        );


    const errorElement =
        document.getElementById(
            "percentageError"
        );


    const statusElement =
        document.getElementById(
            "validationStatus"
        );


    if (
        !calculatedElement ||
        !experimentalElement ||
        !errorElement ||
        !statusElement
    ) {

        return;

    }


    const calculated =
        parseFloat(
            calculatedElement.textContent
        );


    const experimental =
        parseFloat(
            experimentalElement.value
        );


    if (
        isNaN(experimental) ||
        experimental <= 0
    ) {

        errorElement.textContent =
            "—";


        statusElement.textContent =
            "Waiting for reference value";


        statusElement.className =
            "validation-status";

        return;

    }


    const error =
        Math.abs(
            (calculated - experimental) /
            experimental
        ) * 100;


    errorElement.textContent =
        error.toFixed(2);


    if (error <= 5) {

        statusElement.textContent =
            "✓ GOOD AGREEMENT";


        statusElement.className =
            "validation-status success";

    }

    else if (error <= 10) {

        statusElement.textContent =
            "⚠ ACCEPTABLE DIFFERENCE";


        statusElement.className =
            "validation-status warning";

    }

    else {

        statusElement.textContent =
            "✕ HIGH DIFFERENCE";


        statusElement.className =
            "validation-status danger";

    }

}


/* =========================================================
   CANVAS GYROSCOPE
========================================================= */

function drawGyroscope() {

    const canvas =
        document.getElementById(
            "wheelCanvas"
        );


    if (!canvas) {
        return;
    }


    const ctx =
        canvas.getContext("2d");


    const width =
        canvas.width;


    const height =
        canvas.height;


    /* Clear */

    ctx.clearRect(
        0,
        0,
        width,
        height
    );


    /* -----------------------------------------------------
       Background
    ----------------------------------------------------- */

    const gradient =
        ctx.createRadialGradient(
            width / 2,
            height / 2,
            20,
            width / 2,
            height / 2,
            350
        );


    gradient.addColorStop(
        0,
        "rgba(40, 120, 255, 0.10)"
    );


    gradient.addColorStop(
        1,
        "rgba(0, 0, 0, 0)"
    );


    ctx.fillStyle =
        gradient;


    ctx.fillRect(
        0,
        0,
        width,
        height
    );


    /* -----------------------------------------------------
       Center
    ----------------------------------------------------- */

    const cx =
        width / 2;


    const cy =
        height / 2;


    const radius =
        Math.min(
            width,
            height
        ) * 0.25;


    /* -----------------------------------------------------
       Precession orbit
    ----------------------------------------------------- */

    ctx.save();


    ctx.strokeStyle =
        "rgba(90, 160, 255, 0.25)";


    ctx.lineWidth = 2;


    ctx.setLineDash([
        7,
        7
    ]);


    ctx.beginPath();


    ctx.ellipse(
        cx,
        cy,
        radius * 1.55,
        radius * 0.65,
        0,
        0,
        Math.PI * 2
    );


    ctx.stroke();


    ctx.setLineDash([]);


    ctx.restore();


    /* -----------------------------------------------------
       Rotating wheel
    ----------------------------------------------------- */

    ctx.save();


    ctx.translate(
        cx,
        cy
    );


    ctx.rotate(
        animationAngle
    );


    /* Outer wheel */

    ctx.beginPath();


    ctx.ellipse(
        0,
        0,
        radius,
        radius * 0.34,
        0,
        0,
        Math.PI * 2
    );


    ctx.strokeStyle =
        "rgba(80, 160, 255, 0.95)";


    ctx.lineWidth = 8;


    ctx.stroke();


    /* Inner wheel */

    ctx.beginPath();


    ctx.ellipse(
        0,
        0,
        radius * 0.72,
        radius * 0.24,
        0,
        0,
        Math.PI * 2
    );


    ctx.strokeStyle =
        "rgba(100, 180, 255, 0.45)";


    ctx.lineWidth = 3;


    ctx.stroke();


    /* Spokes */

    for (
        let i = 0;
        i < 8;
        i++
    ) {

        const angle =
            (Math.PI * 2 * i) / 8;


        const x =
            Math.cos(angle) *
            radius;


        const y =
            Math.sin(angle) *
            radius *
            0.34;


        ctx.beginPath();


        ctx.moveTo(
            0,
            0
        );


        ctx.lineTo(
            x,
            y
        );


        ctx.strokeStyle =
            "rgba(120, 190, 255, 0.45)";


        ctx.lineWidth = 2;


        ctx.stroke();

    }


    /* Hub */

    ctx.beginPath();


    ctx.arc(
        0,
        0,
        radius * 0.12,
        0,
        Math.PI * 2
    );


    ctx.fillStyle =
        "rgba(100, 180, 255, 1)";


    ctx.fill();


    ctx.restore();


    /* -----------------------------------------------------
       Spin axis
    ----------------------------------------------------- */

    ctx.save();


    ctx.translate(
        cx,
        cy
    );


    ctx.rotate(
        -0.45
    );


    ctx.beginPath();


    ctx.moveTo(
        -radius * 1.5,
        0
    );


    ctx.lineTo(
        radius * 1.5,
        0
    );


    ctx.strokeStyle =
        "rgba(70, 170, 255, 0.75)";


    ctx.lineWidth = 2;


    ctx.stroke();


    /* Arrow */

    drawArrow(
        ctx,
        radius * 1.5,
        0,
        radius * 1.15,
        0
    );


    ctx.restore();


    /* -----------------------------------------------------
       Precession axis
    ----------------------------------------------------- */

    ctx.beginPath();


    ctx.moveTo(
        cx,
        cy - radius * 1.55
    );


    ctx.lineTo(
        cx,
        cy + radius * 1.55
    );


    ctx.strokeStyle =
        "rgba(120, 220, 255, 0.55)";


    ctx.lineWidth = 2;


    ctx.stroke();


    /* Precession arrow */

    drawArrow(
        ctx,
        cx,
        cy - radius * 1.55,
        cx,
        cy - radius * 1.15
    );


    /* -----------------------------------------------------
       Labels
    ----------------------------------------------------- */

    ctx.font =
        "bold 15px Arial";


    ctx.fillStyle =
        "rgba(130, 200, 255, 0.95)";


    ctx.fillText(
        "ω  Spin",
        cx + radius * 1.25,
        cy - radius * 0.9
    );


    ctx.fillStyle =
        "rgba(150, 220, 255, 0.95)";


    ctx.fillText(
        "ωₚ  Precession",
        cx + 15,
        cy - radius * 1.4
    );


    ctx.fillStyle =
        "rgba(255, 255, 255, 0.8)";


    ctx.font =
        "13px Arial";


    ctx.fillText(
        "Rotating Wheel",
        cx - 48,
        cy + radius * 1.4
    );

}


/* =========================================================
   DRAW ARROW
========================================================= */

function drawArrow(
    ctx,
    fromX,
    fromY,
    toX,
    toY
) {

    const headLength = 12;


    const angle =
        Math.atan2(
            toY - fromY,
            toX - fromX
        );


    ctx.beginPath();


    ctx.moveTo(
        toX,
        toY
    );


    ctx.lineTo(
        toX -
        headLength *
        Math.cos(angle - Math.PI / 6),

        toY -
        headLength *
        Math.sin(angle - Math.PI / 6)
    );


    ctx.moveTo(
        toX,
        toY
    );


    ctx.lineTo(
        toX -
        headLength *
        Math.cos(angle + Math.PI / 6),

        toY -
        headLength *
        Math.sin(angle + Math.PI / 6)
    );


    ctx.stroke();

}


/* =========================================================
   ANIMATION
========================================================= */

function startAnimation() {

    const speed =
        getNumber("wheelSpeed");


    animationAngle +=
        0.005 *
        Math.max(
            speed / 500,
            0.5
        );


    drawGyroscope();


    requestAnimationFrame(
        startAnimation
    );

}


/* =========================================================
   SCROLL
========================================================= */

function scrollToSection(id) {

    const section =
        document.getElementById(id);


    if (!section) {
        return;
    }


    section.scrollIntoView({

        behavior: "smooth",

        block: "start"

    });

}


/* =========================================================
   KEYBOARD SHORTCUTS
========================================================= */

document.addEventListener(
    "keydown",
    function (event) {

        /* R = reset */

        if (
            event.key.toLowerCase() === "r" &&
            !isTyping()
        ) {

            resetInputs();

        }


        /* 1 = speed graph */

        if (
            event.key === "1" &&
            !isTyping()
        ) {

            showGraph("speed");

        }


        /* 2 = precession graph */

        if (
            event.key === "2" &&
            !isTyping()
        ) {

            showGraph("precession");

        }


        /* 3 = mass graph */

        if (
            event.key === "3" &&
            !isTyping()
        ) {

            showGraph("mass");

        }


        /* 4 = radius graph */

        if (
            event.key === "4" &&
            !isTyping()
        ) {

            showGraph("radius");

        }

    }
);


/* =========================================================
   CHECK IF USER IS TYPING
========================================================= */

function isTyping() {

    const active =
        document.activeElement;


    if (!active) {
        return false;
    }


    return (

        active.tagName === "INPUT" ||

        active.tagName === "TEXTAREA" ||

        active.tagName === "SELECT"

    );

}


/* =========================================================
   HANDLE WHEEL TYPE CHANGE
========================================================= */

const wheelTypeElement =
    document.getElementById(
        "wheelType"
    );


if (wheelTypeElement) {

    wheelTypeElement.addEventListener(
        "change",
        function () {

            calculate();

        }
    );

}


/* =========================================================
   HANDLE WINDOW RESIZE
========================================================= */

window.addEventListener(
    "resize",
    function () {

        drawGyroscope();

    }
);