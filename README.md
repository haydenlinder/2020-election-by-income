# 2020-election-by-income

Income data: https://www.ers.usda.gov/data-products/county-level-data-sets/download-data/

Election data: https://embeds.ddhq.io/api/v2/2020presidential

Here's some pseudo-code for how I merged the two data sets:

```js
let data = '/income-data.json' // I used csvjson.com to convert the csv data to json
const filtered = data.filter(datum => datum.Attribute === 'Median_Household_Income_2019')

filtered.forEach(item => {
    const ab = item.Stabr
    let sliceIdx = item.area_name.lastIndexOf(',')
    sliceIdx = sliceIdx === -1 ? item.area_name.length : sliceIdx
    const county = item.area_name.slice(0, sliceIdx)
    const value = { income: item.Value, county, state: ab }
    states[ab] ? states[ab].push(value) : states[ab] = [value]
})

data = '/election-data.json'
data.data.forEach(state => {
    const ab = state.abb
    const counties = state.countyResults.counties

    counties.forEach(county => {
        const name = county.county
        states[ab]?.forEach(c => {
            if (c.county.startsWith(name)) {
                c.biden = county.votes[11918]
                c.trump = county.votes[5]
            }
        })
    })

})

const flat = Object.values(states).flat().filter(data => data.biden || data.trump)

var blob = new Blob([JSON.stringify(flat)], { type: "application/json" });
var url = URL.createObjectURL(blob);
setFileUrl(url)
```