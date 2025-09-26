# Swap-deployment-slots-in-Azure-App-Service
This project, I deploy a static HTML Mini Game website to Azure App Service, create a staging deployment slot, make changes to the code and deploy them to the staging slot, and then swap the staging and production slots to promote the changes to production. I learn how to use deployment slots for safe application updates and blue-green deployments.



Tasks performed in this exercise:
1. Download and deploy the sample app to Azure App Service.
2. Create a staging deployment slot.
3. Make a change to the sample app and deploy it to the staging slot.
4. Swap the staging and default production slots to move the changes to the production slot.


#Download and deploy the sample app
In this section, you download the sample app and set variables to make the commands easier to enter. 
and then create an Azure App Service resource and deploy a static HTML site using Azure CLI commands.

>>> In your browser, navigate to the Azure portal https://portal.azure.com, signing in with your Azure credentials if prompted.

Use the [>_] button to the right of the search bar at the top of the page to create a new Cloud Shell in the Azure portal, selecting a Bash environment. The cloud shell provides a command-line interface in a pane at the bottom of the Azure portal. If you are prompted to select a storage account to persist your files, select No storage account required, your subscription, and then select Apply.

Note: If you have previously created a cloud shell that uses a PowerShell environment, switch it to Bash.

In the cloud shell toolbar, in the Settings menu, select Go to Classic version (this is required to use the code editor).

Run the following git command to clone the sample app repository.

#bash
>>> git clone https://github.com/Azure-Samples/html-docs-hello-world.git

Set variables to hold the resource group and app names by running the following commands. You can replace the rg-mywebapp value for resourceGroup if you have a resource group you want to use. Make note of the value of the appName that is displayed after the commands run, you'll need it later in this exercise.

#bash
>>> resourceGroup=rg-mywebapplod55024252
>> appName=mywebapp55024252
>> echo $appName

Navigate to the directory that contains the sample code and run the az webapp up command. Note: This command might take a few minutes to run.

#bash
>>> cd html-docs-hello-world
>> az webapp up -g $resourceGroup -n $appName --sku P0V3 --html

Now that your deployment has finished it's time to view the web app.

>>> In the Azure portal navigate to the web app you deployed. You can enter the name you noted earlier in the Search resources, services, and docs (G + /) search bar, and select the resource from the list.

1. Select the link to your web app located in the Default domain field in the Essentials section. The link will open the site in a new tab.

>>> Deploy updated code to a deployment slot

In this section you create a deployment slot, modify the HTML in the app, and deploy the updated code to the new deployment slot.

1. Create a deployment slot
2. Return to the tab with the Azure portal and cloud shell.

Enter the following command in the cloud shell to create a deployment slot named staging.

#bash
>>> az webapp deployment slot create -n $appName -g $resourceGroup --slot staging

Wait for the command to finish, and then select Deployment > Deployment slots in the left menu to view the deployment slots for your web app. Note the name of the new slot contains -staging appended to name of your web app

Update code and deploy to the staging slot
#In the cloud shell, 
#bash 
index.html to open the editor. 
Locate the <h1> heading tag, and change Azure App Service - Sample Static HTML Site to Azure App Service Staging Slot - or to anything else that you'd like.

#We will build a mini Game

<!DOCTYPE html>
<html>
<head>
<title>Memory Game</title>
<style>
body{font-family:Arial;text-align:center;background:#f0f8ff;margin:40px}
.board{display:grid;grid-template-columns:repeat(4,80px);gap:10px;justify-content:center;margin:20px}
.card{width:80px;height:80px;background:#4a6fa5;color:white;font-size:24px;display:flex;
align-items:center;justify-content:center;cursor:pointer;border-radius:8px;transition:all 0.3s}
.card.flipped{background:#ff6b6b;transform:rotateY(180deg)}
</style>
</head>
<body>
<h1>🧠 Memory Game</h1>
<p>Find all matching pairs!</p>
<div class="board" id="board"></div>
<p>Moves: <span id="moves">0</span></p>
<button onclick="resetGame()">New Game</button>

<script>
const emojis = ['🐶','🐱','🐭','🐹','🐰','🦊','🐻','🐼'];
let cards = [], flippedCards = [], moves = 0, matches = 0;

function initGame() {
    cards = [...emojis, ...emojis].sort(() => Math.random() - 0.5);
    document.getElementById('board').innerHTML = cards.map((emoji, i) => 
        `<div class="card" onclick="flipCard(${i})"></div>`).join('');
    moves = matches = 0; updateMoves();
}

function flipCard(index) {
    const card = document.getElementsByClassName('card')[index];
    if (flippedCards.length < 2 && !card.classList.contains('flipped')) {
        card.classList.add('flipped');
        card.textContent = cards[index];
        flippedCards.push({index, emoji: cards[index]});
        
        if (flippedCards.length === 2) {
            moves++; updateMoves();
            if (flippedCards[0].emoji === flippedCards[1].emoji) {
                matches++; flippedCards = [];
                if (matches === emojis.length) setTimeout(() => alert(`You won in ${moves} moves!`), 300);
            } else {
                setTimeout(() => {
                    flippedCards.forEach(f => {
                        document.getElementsByClassName('card')[f.index].classList.remove('flipped');
                        document.getElementsByClassName('card')[f.index].textContent = '';
                    });
                    flippedCards = [];
                }, 1000);
            }
        }
    }
}

function updateMoves() { document.getElementById('moves').textContent = moves; }
function resetGame() { initGame(); }
initGame();
</script>
</body>
</html>

Use the commands ctrl-s to save, and ctrl-q to exit.

In the cloud shell run the following command to create a zip file of the updated project. A zip, or a web application resource (WAR), file is needed for the next step.

#bash
>>> zip -r stagingcode.zip .

Run the following command in the cloud shell to deploy your updates to the staging slot.

#bash
>>> az webapp deploy -g $resourceGroup -n $appName --src-path ./stagingcode.zip --slot staging

Select Deployment > Deployment slots in the left menu of your web app, and then select the staging slot you created earlier.

Select the link in the Default domain field in the Essentials section. The link will open the web site for the staging slot in a new tab.

#Swap the staging and production slots
1. You can perform a swap in the Azure portal with the Swap option in the toolbar. 
2. The Swap option will appear in the toolbar if you select Overview or Deployment > Deployment slots in the left menu of your web app.

#In the Azure portal, select Swap in the toolbar to open the Swap panel.

1. Review the settings in the swap panel. The Source should show the -staging slot, and the Target should show the default production slot.

2. Select Start Swap and wait for the operation to complete. You can track completion in the Notifications panel that you can open by selecting the bell icon at the top of the portal.

3. To verify the swap, navigate to the web app you deployed. Enter the web app name you created earlier (for example, mywebapp12360) in the Search resources, services, and docs (G + /) search bar, and then select the resource from the list.

Select the link to your web app located in the Default domain field in the Essentials section. The link will open the site (production slot) in a new tab.

Verify your changes; you may need to refresh the page for them to appear.


Clean up resources
Now that you finished the project, you should delete the cloud resources you created to avoid unnecessary resource usage.

In your browser, navigate to the Azure portal https://portal.azure.com, signing in with your Azure credentials if prompted.
Navigate to the resource group you created and view the contents of the resources used in this exercise.
On the toolbar, select Delete resource group.
Enter the resource group name and confirm that you want to delete it.


CAUTION: Deleting a resource group deletes all resources contained within it. If you chose an existing resource group for this exercise, any existing resources outside the scope of this exercise will also be deleted.

Congratulations
successfully project built
