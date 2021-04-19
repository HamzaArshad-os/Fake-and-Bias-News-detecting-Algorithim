# Fake-and-Bias-News-detecting-Algorithim


using Microsoft.ML;
using Microsoft.ML.Data;
using System;
using System.Collections.Generic;
using System.Collections.Specialized;
using System.IO;
using System.Linq;
using System.Net;
using System.Text;
using System.Text.RegularExpressions;
using static Microsoft.ML.DataOperationsCatalog;
using static System.Net.Mime.MediaTypeNames;

namespace SentimentAnalysis
{


    class Program
    {
        string urlAddressA = @""; //The URL of the article the user is reading from 
        string urlAddress = @"";
        string BaseHTML = "";//All the HTML of the article the user is reading is placed here
        bool TitlestartPresnt = false;  //Set of checks to verify that  a title exsits in the articles
        bool TitleendPresent = false;
        bool TitlestartPresntC = false;
        bool TitleendPresentC = false;


        int PositiveTitleB = 0; //Will be either 1 or 0
        int NegaitiveTitleB = 0;  //Will be either 1 or 0
        int TotalNumofWordsofB = 0;  //For Comparison later
        string Intermeddiate = ""; //The purifed version of each article is placed here for the next set of checks or methods tobe ran on.
        string Title = ""; // The title of the article the user is reading is placed here

        List<string> TitlesC = new List<string>();
        string Titlestart = $"<title>";  //For locating the title of the articles
        string Titleend = $"</title>";


        string PureBasetext2 = "";
        Dictionary<string, int> statsB = new Dictionary<string, int>();  //For counting the words
        Dictionary<string, int> statsC = new Dictionary<string, int>();// The word is stored a sa stirng and the count as a integer
        List<string> QouteListB = new List<string>();
        int NumofQoutesB = 0;
        int LengthofB = 0;
        int LenghofCAll = 1;
        List<string> QoutesofB = new List<string>();
        List<int> CATEGORISEDDateTimes = new List<int>(); // Setup but never used as did not work
        List<int> CurrentDateTime = new List<int>();// Setup but never used as did not work
        List<int> BASEDATETIME = new List<int>();// Setup but never used as did not work
        List<string> ListofDTofC = new List<string>();// Setup but never used as did not work
        List<int> CATEGORISEDDateTimesC = new List<int>();// Setup but never used as did not work
        List<string> URLSofCs = new List<string>(); //All URLS of the comparison articles are stored here before verification
        List<string> HtmlofEachC = new List<string>();
        List<string> PurifedEachC = new List<string>();
        List<string> QouteListC = new List<string>();
        List<string> WordsforSpellCheck = new List<string>();
        List<string> RelevantWordsfromB = new List<string>();
        List<string> RelevantWordsfromC = new List<string>();
        List<string> PurifedDTMC = new List<string>();
        List<string> TitlesCList = new List<string>();
        int URLCount = 0;
        List<string> PurestC2 = new List<string>();
        List<string> DTofeachC = new List<string>();
        List<int> DateTimeofEachC = new List<int>();
        List<int> lengthofeachC = new List<int>();
        int NumofQOutesC = 0;

        static readonly string _dataPath = Path.Combine(Environment.CurrentDirectory, "Data", "yelp_labelled.txt");//Required for Sentimental Analysis
        List<string> SentiInputSentences = new List<string>();
        int totalPositive = 0;  //Implemented from the results of Sentimental analysis 
        int totalNegative = 0;
        int totalPositiveC = 0;
        int totalNegativeC = 0;
        int totalPositiveCAVG = 0;
        int totalNegativeCAVG = 0;
        int PositiveTitles = 0;
        int NegativeTitles = 0; //Implemented from the results of Sentimental analysis 
        List<string> Links = new List<string>();// The  links are stored here.

        bool BasePosOrneg = false; //False = Negative  True = Positive
        bool CPosOrNeg = false;


        int TotalSamewords = 0;


        double ProbabilityofBiengBiast = 0; //Used throughout the program to give final results
        double ProbabilityofFakeNews = 0; //Used throughout the program to give final results


        double ProbabilityofBiengBiastC = 0; //Used throughout the program to give final results
        double ProbabilityofFakeNewsC = 0; //Used throughout the program to give final results

        bool URLCountcheck = false;
        string urlAddressC = "";// The search reuqest during the Webcrawling process.
        int TitleWordCount = 0;
        List<string> RoundsFakeNews = new List<string>();
        List<string> RoundsBias = new List<string>();
        List<string> TriedURLS = new List<string>(); //For filtering out URLs that can not be used 
        List<string> Wordsthathaveranthroughspellcheck = new List<string>();


        //TestMode globals
        List<string> validurl = new List<string>();// All valid URLs are stored here
        List<string> URLS = new List<string>(); //All the URLs from the dataset are stored here
        List<string> Titles = new List<string>(); //Title of the article the user is reading is stored here
        List<string> TRUEORFALSE = new List<string>();// Expected Result of the article of each URL in the dataset
        string expectedresult = "";
        int ValidURLCount = 0;
        int ValidTitleCount = 0;
        int ValidBoolCount = 0;
        double GotRight = 0;
        double GotWrong = 0;
        int True = 0;
        int False = 0;

        List<string> Corrects = new List<string>(); //Where program got the expected result 
        List<string> Incorrects = new List<string>();//Where program did not get the expected result 
        int Beta = 0; //Helps order the URLs
        bool Delta = false;

        List<string> Unknowns = new List<string>(); //Any invalid URLs are stored here

        public Program()
        {
            Console.WriteLine("Press Y for TestMode , Press N for Normal Mode"); //Easy way to implement TestMode
            string lol = Console.ReadLine();
            if (lol == "Y" || lol == "y")
            {
                TestMode();
                // ProgramcallerforTester();
            }
            if (lol == "N" || lol == "n")
            {



                Console.BackgroundColor = ConsoleColor.DarkBlue;

                Console.BackgroundColor = ConsoleColor.DarkBlue;
                Console.WriteLine("Examples of articles");
                Console.WriteLine();//Some example Websites
                Console.BackgroundColor = ConsoleColor.DarkGreen;
                Console.WriteLine("Biast News article 1");
                Console.WriteLine("Alexandria Ocasio-Cortez reminds workers to check paystubs following Daylight Saving Time");
                Console.WriteLine("https://www.independent.co.uk/news/world/americas/us-politics/aoc-daylight-saving-time-reminder-pay-stub-hour-extra-workers-a9185101.html");
                Console.WriteLine();
                Console.WriteLine("Fake News article 2");
                Console.WriteLine("Sniffing Rosemary Can Increase Memory by 75%");
                Console.WriteLine("https://dailyhealthpost.com/rosemary-increase-memory/");
                Console.WriteLine();
                Console.WriteLine("Not fake article");
                Console.WriteLine("Australia fires: Military to be deployed to help rescue effort");
                Console.WriteLine("https://www.bbc.co.uk/news/world-australia-50956318");
                Console.WriteLine();
                Console.WriteLine("Biast article");
                Console.WriteLine("Heart health: Cutting saturated fat alone doesn't cut it");
                Console.WriteLine("http://edition.cnn.com/2010/HEALTH/03/24/moh.heartmag.saturated.fat/index.html?hpt=Mid");
                Console.WriteLine();
                Console.WriteLine("Not Biast article");
                Console.WriteLine("Happy New Year: Michigan waitress gets $2,020 tip ahead of 2020");
                Console.WriteLine("https://news.sky.com/story/happy-new-year-michigan-waitress-gets-2-020-tip-ahead-of-2020-11898839");
                Console.WriteLine();
                Console.WriteLine("Fake article");
                Console.WriteLine("Banker Leaves One Percent Tip, Tells Waitress to ‘Get a Real Job’ Read More: Banker Leaves One Percent Tip, Tells Waitress to ‘Get a Real Job’");
                Console.WriteLine("https://thefw.com/banker-leaves-one-percent-tip/");
                Console.WriteLine();
                Console.WriteLine("Real article");
                Console.WriteLine("Coronavirus: More Britons evacuated from Wuhan on French flight");
                Console.WriteLine("https://www.bbc.co.uk/news/health-51345279");
                Console.WriteLine();
                Console.WriteLine("Biast article");
                Console.WriteLine("Rape Apologist 'Roosh' Shutting Down Website After Running Out Of Money");
                Console.WriteLine("https://www.huffingtonpost.co.uk/entry/rape-apologist-roosh-shutting-down-misogynist-website-after-running-out-of-money_n_5bb38dbbe4b0ba8bb2116efe?ri18n=true");
                Console.WriteLine();
                Console.WriteLine("Real article");
                Console.WriteLine("Coronavirus: UK tells all Britons to leave China 'if they can'");
                Console.WriteLine("https://www.bbc.co.uk/news/uk-51374056");
                Console.WriteLine();
                Console.WriteLine("Fake article");
                Console.WriteLine("Man faked being deaf and dumb for 62 years to avoid listening to his wife");
                Console.WriteLine("https://worldnewsdailyreport.com/man-faked-being-deaf-and-dumb-for-62-years-to-avoid-listening-to-his-wife/");
                Console.WriteLine();
                Console.WriteLine("Real article");
                Console.WriteLine("Ian Paterson: Surgeon wounded hundreds amid 'culture of denial'");
                Console.WriteLine("https://www.bbc.co.uk/news/uk-england-birmingham-51369881");
                Console.WriteLine();

                Console.WriteLine("Enter the URL of the website of the article you are reading");
                urlAddressA = Console.ReadLine(); //Reuqest url
                Console.BackgroundColor = ConsoleColor.Blue;
                Console.WriteLine();
                Console.WriteLine();
                Console.WriteLine();
                Console.WriteLine("and the title of the article you are reading ");
                Title = Console.ReadLine();//Request title



                //Start of data collection
                HTMLofBExtraction();

                ContentofB();



                //B almost Done

                GettingLinksandhtml();
                ContentsofEachC();
                ComparingNumberofUseofWords();
                CompareTitle();
                ComparingArticles();
                LENGTHChecks();
                SpellCheckofB();
                PointsOutput();

                //Refresh of all variables for other memory usage
                TotalSamewords = 0;
                ProbabilityofBiengBiast = 0;
                ProbabilityofFakeNews = 0;

                ProbabilityofBiengBiastC = 0;
                ProbabilityofFakeNewsC = 0;

                TitlestartPresnt = false;
                TitleendPresent = false;
                TitlestartPresntC = false;
                TitleendPresentC = false;
                totalPositive = 0;
                totalNegative = 0;
                totalPositiveC = 0;
                totalNegativeC = 0;
                totalPositiveCAVG = 0;
                totalNegativeCAVG = 0;
                PositiveTitles = 0;
                NegativeTitles = 0;
                URLCount = 0;
                Wordsthathaveranthroughspellcheck.Clear();
                statsB.Clear();
                statsC.Clear();
                QouteListB.Clear();
                NumofQoutesB = 0;
                LengthofB = 0;
                LenghofCAll = 1;
                QoutesofB.Clear();
                CATEGORISEDDateTimes.Clear();
                CurrentDateTime.Clear();
                BASEDATETIME.Clear();
                ListofDTofC.Clear();
                CATEGORISEDDateTimesC.Clear();
                URLSofCs.Clear();
                HtmlofEachC.Clear();
                PurifedEachC.Clear();
                QouteListC.Clear();
                WordsforSpellCheck.Clear();
                RelevantWordsfromB.Clear();
                RelevantWordsfromC.Clear();
                PurifedDTMC.Clear();
                TitlesCList.Clear();
                ProbabilityofBiengBiast = 0;
                ProbabilityofFakeNews = 0;
                ProbabilityofFakeNewsC = 0;
                ProbabilityofBiengBiastC = 0;
                urlAddressC = "";



                lol = Console.ReadLine();
                if (lol == "1234")
                {

                    Console.WriteLine();
                    Console.WriteLine();
                    Console.WriteLine("Extracted  URLS");
                    foreach (string item in TriedURLS)
                    {

                        Console.WriteLine();
                        Console.WriteLine(item);
                        Console.WriteLine();
                        Console.WriteLine();

                    }

                }
                else
                {


                    Console.ReadLine();
                }




            }



        }



        static void Main(string[] args)
        {
            Console.WriteLine("Hello World!");

            new Program();
            Console.ReadKey();

            Console.ReadLine();
        }

        public void ProgramcallerforTester()
        {


            //INititation of program whith some specifc subroutines designed for  Testing 
            HTMLofBExtraction();
            ContentofBforTester();
            GettingLinksandhtml();
            ContentsofEachC();
            ComparingQoutes();
            ComparingNumberofUseofWords();
            ComparingQoutes();
            CompareTitle();
            ComparingArticles();
            LENGTHChecks();
            SpellCheckofB();
            PointsOutputforTest();






        }


        public void TestMode()
        {


            //Setup of DataSet
            URLS.Add("https://conservativetears.com/2020/01/24/game-show-host-bob-barker-dead-at-98fortune-left-to-trumps-plan-for-usa/"); //URL address
            Titles.Add("Game Show Host Bob Barker Dead At 98;Fortune Left To ‘Trump’s Plan For USA’"); //Title address
            TRUEORFALSE.Add("false"); //Fake  or True address

            URLS.Add("https://www.bbc.co.uk/news/entertainment-arts-51522715"); //URL address
            Titles.Add("Emotional Elton John halts New Zealand gig after pneumonia diagnosis"); //Title address
            TRUEORFALSE.Add("true"); //Fake  or True address

            URLS.Add(" https://dailyhealthpost.com/rosemary-increase-memory/"); //URL address  // https://dailyhealthpost.com/rosemary-increase-memory/
            Titles.Add("Sniffing Rosemary Can Increase Memory by 75%"); //Title address
            TRUEORFALSE.Add("false"); //Fake  or True address

            URLS.Add("https://www.bbc.co.uk/news/uk-england-birmingham-51369881"); //URL address
            Titles.Add("Ian Paterson: Surgeon wounded hundreds amid 'culture of denial'"); //Title address
            TRUEORFALSE.Add("True"); //Fake  or True address

            URLS.Add("https://thefw.com/banker-leaves-one-percent-tip/"); //URL address
            Titles.Add("Banker Leaves One Percent Tip, Tells Waitress to ‘Get a Real Job’ Read More: Banker Leaves One Percent Tip, Tells Waitress to ‘Get a Real Job’"); //Title address
            TRUEORFALSE.Add("false"); //Fake  or True address

            URLS.Add("https://www.bbc.co.uk/news/world-australia-50956318"); //URL address 
            Titles.Add("Australia fires: Military to be deployed to help rescue effort"); //Title address
            TRUEORFALSE.Add("True"); //Fake  or True address

            URLS.Add("https://bitcoinexchangeguide.com/cristiano-ronaldo-makes-cryptocurrency-headlines-with-bitcoin-transfer-payment-possibility/"); //URL address
            Titles.Add("Cristiano Ronaldo Makes Cryptocurrency Headlines with Bitcoin Transfer Payment Possibility"); //Title address
            TRUEORFALSE.Add("false"); //Fake  or True address

            URLS.Add("https://www.bbc.co.uk/news/world-asia-china-51362336"); //URL address
            Titles.Add("Coronavirus: China admits 'shortcomings and deficiencies'"); //Title address
            TRUEORFALSE.Add("True"); //Fake  or True address

            URLS.Add("https://www.bbc.co.uk/news/uk-51374056"); //URL address
            Titles.Add("Coronavirus: UK tells all Britons to leave China 'if they can'"); //Title address
            TRUEORFALSE.Add("True"); //Fake  or True address

            URLS.Add("https://worldnewsdailyreport.com/man-faked-being-deaf-and-dumb-for-62-years-to-avoid-listening-to-his-wife/"); //URL address
            Titles.Add("Man faked being deaf and dumb for 62 years to avoid listening to his wife"); //Title address
            TRUEORFALSE.Add("false"); //Fake  or True address

            URLS.Add("https://www.bbc.co.uk/news/uk-ands-5011"); //URL address https://www.bbc.co.uk/news/uk-england-leeds-51417011 //allos testing to make sure non valid urls can be sorted
            Titles.Add("Yvette Cooper: Knottingley man jailed over threats about MP"); //Title address
            TRUEORFALSE.Add("true"); //Fake  or True address

            URLS.Add("https://fivethirtyeight.com/features/demographics-not-hacking-explain-the-election-results/"); //URL address
            Titles.Add("Demographics, Not Hacking, Explain The Election Results"); //Title address
            TRUEORFALSE.Add("false"); //Fake  or True address

            URLS.Add("http://nationalreport.net/sarah-palin-bans-muslims-entering-bristol-palin/"); //URL address
            Titles.Add("Sarah Palin Bans Muslims from Entering Bristol Palin"); //Title address
            TRUEORFALSE.Add("false"); //Fake  or True address

            URLS.Add("https://www.bbc.co.uk/news/world-europe-51743697"); //URL address
            Titles.Add("Coronavirus: Italy to close all schools as deaths rise"); //Title address
            TRUEORFALSE.Add("true"); //Fake  or True address

            URLS.Add("https://news.sky.com/story/coronavirus-no-deal-brexit-prep-will-be-dusted-off-to-help-businesses-deal-with-outbreak-11949638"); //URL address
            Titles.Add("Coronavirus: No-deal Brexit prep will be 'dusted off to help businesses deal with outbreak' "); //Title address
            TRUEORFALSE.Add("true"); //Fake  or True address

            URLS.Add("https://news.sky.com/story/boris-johnson-sticking-by-priti-patel-but-labour-say-they-have-new-bullying-claims-11949400"); //URL address
            Titles.Add("Boris Johnson 'sticking by' Priti Patel - but Labour say they have new bullying claims "); //Title address
            TRUEORFALSE.Add("true"); //Fake  or True address

            URLS.Add("https://news.sky.com/story/coronavirus-asian-stocks-turn-higher-on-hopes-of-central-bank-help-11947714"); //URL address
            Titles.Add("Coronavirus: Dow Jones records biggest daily points gain after last week's crash "); //Title address
            TRUEORFALSE.Add("true"); //Fake  or True address

            URLS.Add("https://www.dailystar.co.uk/news/latest-news/melania-trump-body-double-uk-16869452"); //URL address
            Titles.Add("Does Melania Trump have a BODY DOUBLE? Photos of First Lady in UK fuels theory"); //Title address
            TRUEORFALSE.Add("false"); //Fake  or True address

            URLS.Add("http://archive.is/0fpXU"); //URL address
            Titles.Add("BREAKING: Roy Moore’s Accuser Arrested And Charged With Falsification"); //Title address
            TRUEORFALSE.Add("false"); //Fake  or True address

            VERIFICATIONIFTESTURLS();


            if (validurl.Count() == Titles.Count() && Titles.Count() == TRUEORFALSE.Count()) //Validation of no errors in dataset so no issues later on
            {

                foreach (string url in validurl)
                {



                    Console.WriteLine("ello governah");
                    if (url.Contains("http"))
                    {
                        urlAddressA = url; //State the current url as the url of the article being processed 
                        Title = Titles[Beta]; //State the current title as the title of the article being processed 
                        expectedresult = TRUEORFALSE[Beta]; //State the current expectedresult  as the expectedresult of the article being processed 

                        URLCount += 1;

                        if (expectedresult == "True" || expectedresult == "TRUE" || expectedresult == "true")
                        {
                            True += 1;
                            ValidBoolCount += 1;


                        }
                        if (expectedresult == "False" || expectedresult == "FALSE" || expectedresult == "false")
                        {
                            False += 1;
                            ValidBoolCount += 1;
                        }
                        //Restarting all vairiables , counts and removing all data from lists , done each time the program is ran in test mode
                        //TotalSamewords = 0;

                        //ProbabilityofBiengBiast = 0;
                        //ProbabilityofFakeNews = 0;

                        //ProbabilityofBiengBiastC = 0;
                        //ProbabilityofFakeNewsC = 0;

                        //TitlestartPresnt = false;
                        //TitleendPresent = false;
                        //TitlestartPresntC = false;
                        //TitleendPresentC = false;
                        //totalPositive = 0;
                        //totalNegative = 0;
                        //totalPositiveC = 0;
                        //totalNegativeC = 0;
                        //totalPositiveCAVG = 0;
                        //totalNegativeCAVG = 0;
                        //PositiveTitles = 0;
                        //NegativeTitles = 0;
                        //URLCount = 0;
                        //statsB.Clear();
                        //statsC.Clear();
                        //QouteListB.Clear();
                        //NumofQoutesB = 0;
                        //LengthofB = 0;
                        //LenghofCAll = 1;
                        //QoutesofB.Clear();
                        //CATEGORISEDDateTimes.Clear();
                        //CurrentDateTime.Clear();
                        //BASEDATETIME.Clear();
                        //ListofDTofC.Clear();
                        //CATEGORISEDDateTimesC.Clear();
                        //URLSofCs.Clear();
                        //HtmlofEachC.Clear();
                        //PurifedEachC.Clear();
                        //QouteListC.Clear();
                        //WordsforSpellCheck.Clear();
                        //RelevantWordsfromB.Clear();
                        //RelevantWordsfromC.Clear();
                        //PurifedDTMC.Clear();
                        //TitlesCList.Clear();
                        //urlAddressC = "";
                        //Wordsthathaveranthroughspellcheck.Clear();




                        urlAddress = @"";
                        BaseHTML = "";
                        TitlestartPresnt = false;
                        TitleendPresent = false;
                        TitlestartPresntC = false;
                        TitleendPresentC = false;


                        PositiveTitleB = 0;
                        NegaitiveTitleB = 0;
                        TotalNumofWordsofB = 0;
                        Intermeddiate = "";


                        TitlesC.Clear();
                        Titlestart = $"<title>";
                        Titleend = $"</title>";
                        PureBasetext2 = "";
                        statsB.Clear();
                        statsC.Clear();
                        QouteListB.Clear();
                        NumofQoutesB = 0;
                        LengthofB = 0;
                        LenghofCAll = 1;
                        QoutesofB.Clear();
                        CATEGORISEDDateTimes.Clear();
                        CurrentDateTime.Clear();
                        BASEDATETIME.Clear();
                        ListofDTofC.Clear();
                        CATEGORISEDDateTimesC.Clear();
                        URLSofCs.Clear();
                        HtmlofEachC.Clear();
                        PurifedEachC.Clear();
                        QouteListC.Clear();
                        WordsforSpellCheck.Clear();
                        RelevantWordsfromB.Clear();
                        RelevantWordsfromC.Clear();
                        RelevantWordsfromC.Clear();
                        PurifedDTMC.Clear();
                        TitlesCList.Clear();
                        URLCount = 0;
                        PurestC2.Clear();
                        DTofeachC.Clear();
                        DateTimeofEachC.Clear();
                        lengthofeachC.Clear();
                        NumofQOutesC = 0;
                        SentiInputSentences.Clear();
                        totalPositive = 0;
                        totalNegative = 0;
                        totalPositiveC = 0;
                        totalNegativeC = 0;
                        totalPositiveCAVG = 0;
                        totalNegativeCAVG = 0;
                        PositiveTitles = 0;
                        NegativeTitles = 0;
                        Links.Clear();
                        BasePosOrneg = false;
                        CPosOrNeg = false;
                        TotalSamewords = 0;
                        ProbabilityofBiengBiast = 0;
                        ProbabilityofFakeNews = 0;
                        ProbabilityofBiengBiastC = 0;
                        ProbabilityofFakeNewsC = 0;
                        URLCountcheck = false;
                        urlAddressC = "";
                        TitleWordCount = 0;
                        RoundsFakeNews.Clear();
                        RoundsBias.Clear();
                        TriedURLS.Clear();
                        Wordsthathaveranthroughspellcheck.Clear();




                        ProgramcallerforTester();

                        // urlAddress = validurl[+1];
                        //Title = Titles[+1];
                        //expectedresult = TRUEORFALSE[+1];

                        ValidTitleCount += 1;
                        Beta += 1;

                    }
                    else
                    {
                        Console.WriteLine("Not a http contaning website");
                        Unknowns.Add(url);
                    }



                }
                //Calls final percentage reuslts
                Percents();

            }
            else
            {
                Console.WriteLine("Different number of urls to titles  to trueorfalse");

            }





        }


        public void VERIFICATIONIFTESTURLS()
        {

            foreach (string url in URLS)
            {
                try
                {

                    HttpWebRequest request = (HttpWebRequest)WebRequest.Create(url);   //Validation the websites added are real.
                    HttpWebResponse response = (HttpWebResponse)request.GetResponse();
                    validurl.Add(url);
                }
                catch (Exception ex)
                {
                    Console.WriteLine(ex);
                    Console.WriteLine("Invalid url"); //If not remove from the index of it from all 3 lists
                    int index = URLS.IndexOf(url);
                    //URLS.Remove(url);
                    Titles.RemoveAt(index);
                    TRUEORFALSE.RemoveAt(index);
                    Unknowns.Add(url);

                }



            }
        }

        public void HTMLofBExtraction()
        {
            if (!urlAddressA.Contains("Reddit") && !urlAddressA.Contains("Qoura")) //Harder to extract valuable data from sources like this.
            {
                try
                {
                    HttpWebRequest request = (HttpWebRequest)WebRequest.Create(urlAddressA);   // collects html from this website by first making a request
                    HttpWebResponse response = (HttpWebResponse)request.GetResponse();
                    if (response.StatusCode == HttpStatusCode.OK) //if sussesful request by a return
                    {
                        Stream receiveStream = response.GetResponseStream();  //collects html
                        StreamReader readStream = null;
                        if (response.CharacterSet == null)
                        {
                            readStream = new StreamReader(receiveStream);
                            Console.WriteLine("Sorry , somethings faulty");


                        }
                        else
                        {
                            readStream = new StreamReader(receiveStream, Encoding.GetEncoding(response.CharacterSet));
                            readStream.ToString();
                            // Console.WriteLine(readStream);
                        }
                        string ImpureTexta = readStream.ReadToEnd().ToString();
                        BaseHTML = ImpureTexta; //extracting the entire html collected to a global variable to be used by the program
                        Console.WriteLine(BaseHTML);

                        Console.WriteLine(" <--------------------------------Extraction  B Complete --------------------------------->");
                        Console.WriteLine();
                    }
                }
                catch (Exception EX)
                {
                    Console.WriteLine(EX);
                    Console.WriteLine("Yoink");
                    Console.WriteLine("Could not find the article you are reading , unable to do tests");
                    Console.ReadKey();


                }

            }

        }
        public void ContentofB()
        {


            MLContext mlContext = new MLContext(); //Setup of Sentimental analysis 
            TrainTestData splitDataView = LoadData(mlContext);
            ITransformer model = BuildAndTrainModel(mlContext, splitDataView.TrainSet);  // preperation, training , evaluation done.
            Evaluate(mlContext, model, splitDataView.TestSet);
            UseModelWithSingleItem(mlContext, model);


            ////Title
            if (BaseHTML.Contains(Titlestart) && BaseHTML.Contains(Titleend))   //Verfies a title is contained in the html 
            {
                TitlestartPresnt = true;
                TitleendPresent = true;


                if (TitlestartPresnt && TitleendPresent)
                {
                    Console.WriteLine();
                    Console.WriteLine("Title Found");
                    Console.WriteLine();
                    ProbabilityofFakeNews -= 40;    //soke fake news articles have lower quality html so it is harder to find the title. 
                    TitleB();  //Extraction of title for later comparison, later found it easier to just reuqest from user.


                }


            }
            else
            {


                ProbabilityofFakeNews += 40; //the program scrutinises this by adding points to likeliness of being fake

                TitlestartPresnt = false;
                TitleendPresent = false;
            }





            if (urlAddressA.Contains("https"))   //https is secure. so higher chance of being a not fake source
            {
                //Points
                ProbabilityofFakeNews -= 200;

                ProbabilityofBiengBiast -= 20;
            }
            else
            {
                //Points redueced
                ProbabilityofFakeNews += 200;

                ProbabilityofBiengBiast += 20;
            }


            CompressionofB();
            CountingNumofWordsB();
            // QoutesExtractionofB();


            TitlestartPresnt = false;
            TitleendPresent = false;

            SentiInput();  //Extracts all Sentences to be placed into Senti  

            UseModelWithBatchItems(mlContext, model, SentiInputSentences);
            UseModelWithBatchItemsBTitle(mlContext, model, SentiInputSentences);
            Intermeddiate = "";

        }//SENTIOFB and SENTIOFTITLE

        public void ContentofBforTester()
        {
            //More or less the same thing with just a few tweals
            MLContext mlContext = new MLContext();
            TrainTestData splitDataView = LoadData(mlContext);
            ITransformer model = BuildAndTrainModel(mlContext, splitDataView.TrainSet);
            Evaluate(mlContext, model, splitDataView.TestSet);
            UseModelWithSingleItem(mlContext, model);


            ////Title
            if (BaseHTML.Contains(Titlestart) && BaseHTML.Contains(Titleend))
            {
                TitlestartPresnt = true;
                TitleendPresent = true;

                if (TitlestartPresnt && TitleendPresent)
                {
                    Console.WriteLine();
                    Console.WriteLine("Title Found");
                    Console.WriteLine();
                    ProbabilityofFakeNews -= 40;
                    TitleBForTester();


                }


            }
            else
            {


                ProbabilityofFakeNews += 40;

                TitlestartPresnt = false;
                TitleendPresent = false;
            }





            if (urlAddressA.Contains("https"))
            {
                //Points
                ProbabilityofFakeNews -= 200;

                ProbabilityofBiengBiast -= 20;
            }
            else
            {
                //Points redueced
                ProbabilityofFakeNews += 200;

                ProbabilityofBiengBiast += 20;
            }


            CompressionofB();
            CountingNumofWordsB();
            // QoutesExtractionofB();


            TitlestartPresnt = false;
            TitleendPresent = false;

            SentiInput();  //Extracts all Sentences to be placed into Senti  

            UseModelWithBatchItems(mlContext, model, SentiInputSentences);    //Does senti anaylis on sentences of the article the user is reading
            UseModelWithBatchItemsBTitle(mlContext, model, SentiInputSentences);  //Does senti anaylis on title of the article the user is reading
            Intermeddiate = ""; //restarting intermeddiate..

        }//SENTIOFB and SENTIOFTITLE
        public void CompressionofB()
        {


            //Extracts the data the user is reading
            var matches = Regex.Matches(BaseHTML, "<p>(.*?)</p>");
            string Words = "";
            var irrelevantPWords = new List<string>
                    {
                                 " and ", " is ", " the "," for ", " first ", " on ", " because "," to ", " but "," of ", " was " , " in ","<p>" , "</p>","<p" , "p>","<strong>", "</strong>","~","`","!","#","$","%","^","*","_","+","=","{","}","[","]","|",":","/","&nbsp;", "http"
                      // insert irrelvant words here
                     };
            //removing irrevant words for compression
            //optimisation



            //Removing Irrelvant text
            foreach (var m in matches)
            {


                string PureText = m.ToString();


                foreach (var irrelevantWord in irrelevantPWords)
                {
                    while (PureText.IndexOf(irrelevantWord) > -1)
                    {
                        PureText = PureText.Replace(irrelevantWord, string.Empty + " ");


                    }
                }

                PureText.ToString();



                string[] ssizes = PureText.Split(' ', '\t', '\n');
                foreach (var word in ssizes)
                {
                    string y = word.ToString();
                    if (y.Contains("data-") || y.Contains("href") || y.Contains("span") || y.Contains("target=") || y.Contains("_blank") || y.Contains("rel=") || y.Contains("class="))
                    {

                        word.Replace(y, " ");



                    }
                    else
                    {
                        Words += y;
                        Words += " ";
                        Console.Write(y + " ");
                        RelevantWordsfromB.Add(y);
                    }

                }

            }


            PureBasetext2 = Words;
            Console.WriteLine(Words);
            Console.WriteLine();
            Intermeddiate = PureBasetext2;







            Console.WriteLine("_________________________Compression B Complete____________________________________________");
            Console.WriteLine();
            //Console.WriteLine(PureBasetext2);
            Console.WriteLine("<------------------------------------>");
            Console.WriteLine("<------------------------------------>");
            Console.WriteLine("<------------------------------------>");
            Console.WriteLine("<------------------------------------>");
            Console.WriteLine("<------------------------------------>");
            Console.WriteLine("<------------------------------------>");

            LengthofB = PureBasetext2.Length;

        }
        public void CountingNumofWordsB()
        {
            //Counting Number of each word


            int Count = Intermeddiate.Split(new char[] { ' ', '.', ',', '?' }, StringSplitOptions.RemoveEmptyEntries).Count();
            TotalNumofWordsofB = Count;
            string text = PureBasetext2;
            char[] chars = { ' ', '.', ',', ';', ':', '?', '\n', '\r' };
            // split words
            string[] words = text.Split(chars);

            // iterate over the word collection to count occurrences

            foreach (string word in words)
            {
                string w = word.Trim().ToLower();

                if (!statsB.ContainsKey(w))
                {
                    // add new word to collection
                    statsB.Add(w, 1);

                }
                else
                {
                    // update word occurrence count
                    statsB[w] += 1;
                }


            }




            foreach (var pair in statsB)
            {

                Console.WriteLine("Total occurrences of {0}: {1}", pair.Key, pair.Value);
                if (!statsB.Contains(pair))
                {
                    statsB.Add(pair.Key, pair.Value);
                }



            }



            Console.WriteLine("Number of words = " + Count);


            Console.WriteLine("<------------------------------------------Count B Complete----------------------------->");
        }
        public void SpellCheckofB()
        {

            string[] ssizes = PureBasetext2.Split(' ', '\t', '\n', '.', '!', '?');
            string Words = "";
            foreach (var word in ssizes)
            {
                string y = word.ToString().ToLower();

                if (y == "")
                {
                    y = "calculus"; //Error occurred if y  = ""  so just implemented a word and removed in if statement below 

                }
                if (!y.Contains("data-") || !y.Contains("calculus") || !y.Contains("href") || !y.Contains("span") || !y.Contains("target=") || !y.Contains("_blank") || !y.Contains("rel=") || !y.Contains("class="))
                {

                    word.Replace(y, " ");

                    Words += y;
                    Words += " ";
                    Console.Write(y + " ");
                    //  using (System.IO.StreamWriter file = new System.IO.StreamWriter(@"F:\CompletedCode\CompletedCode\WordstoCompareto.txt", true))


                    if (WordsforSpellCheck.Contains(y))
                    {
                        //Points
                        ProbabilityofFakeNews -= 3;
                        ProbabilityofBiengBiast -= 7;
                        Wordsthathaveranthroughspellcheck.Add(y);

                    }






                }


            }



            Console.WriteLine("<____________________________________Spell Check B Complete ________________________________>");

            Console.WriteLine("All the words that have ran through spell check");
            foreach (string item in Wordsthathaveranthroughspellcheck)
            {
                Console.WriteLine(item);

            }

        }

        public void QoutesExtractionofB()
        {
            Console.WriteLine("Qoutes");

            string b = Intermeddiate.ToString();
            int NumofQOutes = 0;


            string beep = PureBasetext2.Replace("'", "\""); //Setup 
            string beep2 = beep.Replace("&#", "\"") + 4;// Setup of html to quotation marks.


            MatchCollection Qoutes = Regex.Matches(beep2, "\"([^\"]*)\"");  //   (["'&#])(?:(?=(\\?))\2.)*?\1  //Regex to extract quotes


            string[] fields = new string[Qoutes.Count];
            for (int i = 0; i < fields.Length; i++)
            {
                fields[i] = Qoutes[i].Groups[1].Value;
            }


            foreach (string a in fields)
            {
                Console.WriteLine("<---->  " + a + "  <------> WACKA");

                if (!a.Contains("data-") || !a.Contains("href") || !a.Contains("span") || !a.Contains("target=") || !a.Contains("_blank") || !a.Contains("rel=") || !a.Contains("class=") || !a.Contains("href"))
                {
                    if (!QoutesofB.Contains(a))
                    {
                        QoutesofB.Add(a.ToString());
                        NumofQOutes += 1;

                    }

                }


            }
            Console.WriteLine("All Qoutes of B found");
            Console.WriteLine(QoutesofB.Count());
            foreach (string item in QoutesofB)
            {
                Console.WriteLine();
                Console.WriteLine(item);
                Console.WriteLine();
            }


            NumofQoutesB += NumofQOutes;
            if (NumofQOutes < 5)
            {
                ProbabilityofFakeNews += 20;

            }


        }

        public void TitleB()
        {



            var matches = Regex.Matches(BaseHTML, @"(?<=<title[\s\n]*>[\s\n]*)(.(?![\s\n]*</title[\s\n]*>))*");
            //Regex as there can be multiple instances of a title in all the HTML
            foreach (var item in matches)
            {
                string lol = item.ToString();
                Console.WriteLine(lol);
                Console.WriteLine();
                Console.WriteLine();


                //This way works just aswell but sometimes there can be multiple places titles in the HTML so is bad there
                //int titlestart = BaseHTML.IndexOf(Titlestart) + Titlestart.Length;
                //int titleend = BaseHTML.IndexOf(Titleend) - 10; //make so works properly
                //string PureTexttitle = BaseHTML.Substring(titlestart, titleend - titlestart);
                //Title = PureTexttitle;

                //Title Purifying 

                var irrelevantPWords = new List<string>
                    {
                          " and ", " is ", " the "," if "," for ", " first ", " on ", " because "," to ", " but "," of ", " was " , " in ","<p>" , "</p>","<p" , "p>", "</a>" ,"<a>","<strong>", "</strong>"
                        // insert irrelvant words here
                    }; //add more irrelvant words can add stuff like the p> glitch


                foreach (string word in irrelevantPWords)
                {
                    if (Title.Contains(word))
                    {

                        Title = Title.Replace(word, string.Empty + " ");

                    }


                }
            }



            Console.WriteLine(Title);


            TitlestartPresnt = true;
            TitleendPresent = true;



            Console.WriteLine("Your article title ? ");
            Console.WriteLine();
            Console.WriteLine(Title);



            //A failed adaptation that removed the last part of the title as it usally consisted of the Source name eg ...Man flys | -BBCNews
            //if (PureTexttitle.Contains('|'))
            //{
            //   string PureTexttitle2 = PureTexttitle.Substring(titlestart, PureTexttitle.IndexOf('|')-1 );
            //    Console.WriteLine(PureTexttitle2);
            //    Title = PureTexttitle2;

            //}
            //if (PureTexttitle.Contains('-'))
            //{
            //    string PureTexttitle2 = PureTexttitle.Substring(titlestart, PureTexttitle.IndexOf('-')-1);
            //    Console.WriteLine(PureTexttitle2);
            //    Title = PureTexttitle2;

            //}
            //else
            //{

            //   }

            // string PureTexttitle2 = PureTexttitle.Remove('|', PureTexttitle.Length);
            //string PureTexttitle3 = PureTexttitle2.Remove('-', PureTexttitle.Length);
            //Console.WriteLine("Title-" + PureTexttitle3);













        }

        public void TitleBForTester()
        {



            var matches = Regex.Matches(BaseHTML, @"(?<=<title[\s\n]*>[\s\n]*)(.(?![\s\n]*</title[\s\n]*>))*");

            var irrelevantPWords = new List<string>
                    {
                          " and ", " is ", " the "," if "," for ", " first ", " on ", " because "," to ", " but "," of ", " was " , " in ","<p>" , "</p>","<p" , "p>", "</a>" ,"<a>","<strong>", "</strong>"
                        // insert irrelvant words here
                    }; //add more irrelvant words can add stuff like the p> glitch

            foreach (var item in matches)
            {
                string lol = item.ToString();
                Console.WriteLine(lol);
                Console.WriteLine();
                Console.WriteLine();


                //This way works just aswell but sometimes there can be multiple places titles in the HTML so is bad there
                //int titlestart = BaseHTML.IndexOf(Titlestart) + Titlestart.Length;
                //int titleend = BaseHTML.IndexOf(Titleend) - 10; //make so works properly
                //string PureTexttitle = BaseHTML.Substring(titlestart, titleend - titlestart);
                //Title = PureTexttitle;

                //Title Purifying 




                foreach (string word in irrelevantPWords)
                {
                    if (Title.Contains(word))
                    {

                        Title = Title.Replace(word, string.Empty + " ");

                    }


                }
                TitlestartPresnt = true;
                TitleendPresent = true;

            }










            //if (PureTexttitle.Contains('|'))
            //{
            //   string PureTexttitle2 = PureTexttitle.Substring(titlestart, PureTexttitle.IndexOf('|')-1 );
            //    Console.WriteLine(PureTexttitle2);
            //    Title = PureTexttitle2;

            //}
            //if (PureTexttitle.Contains('-'))
            //{
            //    string PureTexttitle2 = PureTexttitle.Substring(titlestart, PureTexttitle.IndexOf('-')-1);
            //    Console.WriteLine(PureTexttitle2);
            //    Title = PureTexttitle2;

            //}
            //else
            //{

            //   }

            // string PureTexttitle2 = PureTexttitle.Remove('|', PureTexttitle.Length);
            //string PureTexttitle3 = PureTexttitle2.Remove('-', PureTexttitle.Length);
            //Console.WriteLine("Title-" + PureTexttitle3);













        }



        public static TrainTestData LoadData(MLContext mlContext)
        {
            IDataView dataView = mlContext.Data.LoadFromTextFile<SentimentData>(_dataPath, hasHeader: false);
            TrainTestData splitDataView = mlContext.Data.TrainTestSplit(dataView, testFraction: 0.2);  //20% of the data in the yelp file is used for testing 
            return splitDataView; //Increase this number, accuracy may decrease as less training data
        }
        public static ITransformer BuildAndTrainModel(MLContext mlContext, IDataView splitTrainSet)
        {
            var estimator = mlContext.Transforms.Text.FeaturizeText(outputColumnName: "Features", inputColumnName: nameof(SentimentData.SentimentText))
            .Append(mlContext.BinaryClassification.Trainers.SdcaLogisticRegression(labelColumnName: "Label", featureColumnName: "Features"));
            Console.WriteLine("=============== Create and Train the Model ===============");
            var model = estimator.Fit(splitTrainSet);
            Console.WriteLine("=============== End of training ===============");
            Console.WriteLine();
            return model;
        }
        public static void Evaluate(MLContext mlContext, ITransformer model, IDataView splitTestSet)
        {
            Console.WriteLine("=============== Evaluating Model accuracy with Test data===============");
            IDataView predictions = model.Transform(splitTestSet);
            CalibratedBinaryClassificationMetrics metrics = mlContext.BinaryClassification.Evaluate(predictions, "Label");
            Console.WriteLine();
            Console.WriteLine("Model quality metrics evaluation");
            Console.WriteLine("--------------------------------");
            Console.WriteLine($"Accuracy: {metrics.Accuracy:P2}");
            Console.WriteLine($"Auc: {metrics.AreaUnderRocCurve:P2}");
            Console.WriteLine($"F1Score: {metrics.F1Score:P2}");
            Console.WriteLine("=============== End of model evaluation ===============");
        }
        private static void UseModelWithSingleItem(MLContext mlContext, ITransformer model)
        {
            PredictionEngine<SentimentData, SentimentPrediction> predictionFunction = mlContext.Model.CreatePredictionEngine<SentimentData, SentimentPrediction>(model);
            SentimentData sampleStatement = new SentimentData
            {
                SentimentText = "This was a very good steak"  // Predicted Positive, returned Postive.
            };
            var resultPrediction = predictionFunction.Predict(sampleStatement);
            Console.WriteLine();
            Console.WriteLine("=============== Prediction Test of model with a single sample and test dataset ===============");

            Console.WriteLine();
            Console.WriteLine($"Sentiment: {resultPrediction.SentimentText} | Prediction: {(Convert.ToBoolean(resultPrediction.Prediction) ? "Positive" : "Negative")} | Probability: {resultPrediction.Probability} ");

            Console.WriteLine("=============== End of Predictions ===============");
            Console.WriteLine();
        }
        public void UseModelWithBatchItems(MLContext mlContext, ITransformer model, List<string> SentiInputSentences)
        {
            int Positive = 0;
            int Negative = 0;
            foreach (string item in SentiInputSentences)
            {
                Console.WriteLine(item);
            }
            foreach (string item in SentiInputSentences)
            {


                //Error Only outputs System.ToString(); //Solved
                string sentence = item.ToString();
                Console.WriteLine(sentence);

                IEnumerable<SentimentData> sentiments = new[]
                {
             
                  //new SentimentData
                  //{
                  //   SentimentText = "I love this spaghetti."
                  //},
                  new SentimentData
                  {
                    SentimentText = sentence
                  }

               };


                IDataView batchComments = mlContext.Data.LoadFromEnumerable(sentiments);

                IDataView predictions = model.Transform(batchComments);

                // Use model to predict whether comment data is Positive (1) or Negative (0).
                IEnumerable<SentimentPrediction> predictedResults = mlContext.Data.CreateEnumerable<SentimentPrediction>(predictions, reuseRowObject: false);

                Console.WriteLine();

                Console.WriteLine("=============== Prediction Test of loaded model with multiple samples ===============");

                foreach (SentimentPrediction prediction in predictedResults)
                {
                    Console.WriteLine($"Sentiment: {prediction.SentimentText} | Prediction: {(Convert.ToBoolean(prediction.Prediction) ? "Positive" : "Negative")} | Probability: {prediction.Probability} ");
                    if (prediction.Probability >= 0.5)
                    {
                        Positive += 1;
                    }
                    else
                    {
                        Negative += 1;
                    }
                }
            }
            Console.WriteLine("=============== End of predictions ===============");
            Console.WriteLine("Num of Positives calculated " + Positive);//number of Postive sentences
            Console.WriteLine("Num of Negatives calculated " + Negative);//Number of Negatvie sentences

            totalPositive = Positive;//Extraction to global variable
            totalNegative = Negative;//Extraction to global variable


        }//ForB

        public void GettingLinksandhtml()
        {
            //the most complex subroutine of the program
            try
            {
                string keypoint = "https://www.bing.com/search?q=";
                string WebsearcHhtml = "";
                urlAddressC = keypoint + Title;  //outputs a string to search
                Console.WriteLine("Hello World!");
                HttpWebRequest request = (HttpWebRequest)WebRequest.Create(urlAddressC);  //request
                HttpWebResponse response = (HttpWebResponse)request.GetResponse();// returns a repsonse
                if (response.StatusCode == HttpStatusCode.OK)
                {
                    Stream receiveStream = response.GetResponseStream();
                    StreamReader readStream = null;
                    if (response.CharacterSet == null)
                    {
                        readStream = new StreamReader(receiveStream);
                        Console.WriteLine("Error");
                    }
                    else
                    {

                        readStream = new StreamReader(receiveStream,
                        Encoding.GetEncoding(response.CharacterSet));
                        readStream.ToString();
                        //Console.WriteLine(readStream);
                    }
                    string ImpureChtml = readStream.ReadToEnd().ToString();
                    WebsearcHhtml = ImpureChtml;

                    Console.WriteLine(ImpureChtml);
                    Console.WriteLine("----------Extracted HTML of Websearch------------------------");
                    Console.WriteLine("----------------------------------");
                    Console.WriteLine("----------------------------------");
                    Console.WriteLine("----------------------------------");









                    bool valid = false;
                    bool valid2 = false;


                    var matches = Regex.Matches(WebsearcHhtml.ToString(), "<a href=(.*?)h=");  //collects all links from the websearch  done above
                    Console.WriteLine();
                    foreach (var lo in matches)
                    {
                        string loll = lo.ToString();
                        //Remvoing the "<a href="  and "h=" that comes with using regex
                        Console.WriteLine();
                        string TesterRemverfront = loll.Remove(0, 9);
                        Console.WriteLine(TesterRemverfront);
                        string TesterRemverback = TesterRemverfront.Remove(TesterRemverfront.Length - 4, 4);
                        Console.WriteLine();
                        Console.WriteLine(TesterRemverback);
                        string testerversion = TesterRemverback;      // forms a purified version of the link 
                        Console.WriteLine(testerversion);

                        try
                        {




                            HttpWebRequest request5 = (HttpWebRequest)WebRequest.Create(testerversion);
                            HttpWebResponse response5 = (HttpWebResponse)request.GetResponse();// returns a repsonse
                            Console.WriteLine("Valid ");                                //verifiying the link works
                            Console.WriteLine(testerversion);
                            valid = true;  //Can be used used in try catch below
                        }
                        catch (Exception)
                        {

                            Console.WriteLine("InValid ");
                            Console.WriteLine(testerversion);
                            valid = false;//Can not be used
                        }


                        try
                        {




                            if (testerversion.Contains("https") && valid && !testerversion.Contains("Reddit") && !testerversion.Contains("reddit") && !testerversion.Contains("Qoura") && !Links.Contains(loll) && !testerversion.Contains("go.microsoft.com") && !testerversion.Contains("pintrest") && !testerversion.Contains("Pintrest") || testerversion.Contains("snopes") || !testerversion.Contains("wiki") || !testerversion.Contains("Youtube") || !testerversion.Contains("twitter") || testerversion.Contains("facebook"))
                            {//Filters out youtube , reddit , qoura and others that are eithe ruseless or invalid.

                                if (testerversion.Contains("https"))
                                {
                                    //Points
                                    ProbabilityofFakeNewsC -= 100;
                                    ProbabilityofBiengBiastC -= 20;
                                }
                                else
                                {
                                    //Points redueced
                                    ProbabilityofFakeNewsC += 100; ProbabilityofBiengBiastC += 20;
                                }



                                Console.WriteLine();
                                Console.WriteLine(testerversion);
                                Console.WriteLine();

                                if (testerversion == urlAddressA || Links.Contains(testerversion))  //checks to make sure the article the user is reading is not used for selp comparison
                                {
                                    Console.WriteLine("This is the article the user is reading");
                                }
                                else
                                {


                                    HttpWebRequest request2 = (HttpWebRequest)WebRequest.Create(testerversion);
                                    HttpWebResponse response2 = (HttpWebResponse)request2.GetResponse();  //collects html of a speicifc website (testerversion)
                                    if (response2.StatusCode == HttpStatusCode.OK)
                                    {
                                        Stream receiveStream2 = response2.GetResponseStream();
                                        StreamReader readStream2 = null;
                                        if (response2.CharacterSet == null)
                                        {
                                            readStream2 = new StreamReader(receiveStream2);
                                            Console.WriteLine("Errorabc"); //make skip this link if error

                                        }
                                        else
                                        {
                                            Links.Add(testerversion); // Adds to links that have been used for comparison
                                            readStream2 = new StreamReader(receiveStream2,
                                            Encoding.GetEncoding(response2.CharacterSet));
                                            readStream2.ToString();
                                            //Console.WriteLine(readStream);
                                            ImpureChtml = readStream2.ReadToEnd().ToString();

                                            HtmlofEachC.Add(ImpureChtml);
                                        }



                                        Console.WriteLine(ImpureChtml);
                                        Console.WriteLine("----------Extracted HTML of Link for Comparison------------------------");
                                        Console.WriteLine("----------------------------------");
                                        Console.WriteLine("----------------------------------");
                                        Console.WriteLine("----------------------------------");


                                        if (ImpureChtml.Contains("snopes") || ImpureChtml.Contains("Snopes"))  //does a snopes check 
                                        {//google and some other search engines have provided a fact check link for dubious internet content to help fight fake news
                                            //using this to our advantage but this is not neccesary for the program to work 
                                            string falsecheckstart = "<h5";
                                            string falsecheckend = "</h5>";

                                            int C = ImpureChtml.IndexOf(falsecheckstart) + falsecheckstart.Length;
                                            int D = ImpureChtml.IndexOf(falsecheckend);
                                            string Snopescheck = ImpureChtml.Substring(C, D - C);
                                            if (Snopescheck.Contains("label-false") || ImpureChtml.Contains("atire"))
                                            {
                                                ProbabilityofFakeNews += 1000000;
                                            }
                                            else
                                            {

                                                ProbabilityofFakeNews -= 1000000;
                                            }
                                            HtmlofEachC.Remove(ImpureChtml);  //removing as the snopes check is not an actual article , will skew witht he average of the comparison articles
                                        }



                                    }


                                }



                            }
                            else
                            {
                                TriedURLS.Add(testerversion);  //to help understand which websites are being flitered out
                                Console.WriteLine("This website not valid");

                            }
                            valid = false;


                        }
                        catch (Exception)
                        {

                            valid = false;
                        }

                    }
                    Console.WriteLine();

                }


            }
            catch (Exception EX)
            {
                Console.WriteLine(EX);
                Console.WriteLine("An error Occured while trying to find Comparing articles");
            }



            if (Links.Count() > 5)  //significantly low amounts of links founds  increases propability of fake news
            {
                ProbabilityofFakeNews -= 200;


            }
            else
            {
                ProbabilityofFakeNews += 200;
                ProbabilityofFakeNewsC -= 200;

            }









            //if link count is less that x amount , probability of fake news = higher 
            //if x amount of reuslts within y amount of time where x< b  probability of fake news = higher

            //make so if contains reddit in url , skip




            int lol = HtmlofEachC.Count();
            URLCount = lol;

            if (URLCount == 0)  //collecting 0 links causes for diffiuclty for rest of program to work , so defualt it to 5 if 0 links are found
            {

                URLCount = 5;
                URLCountcheck = true;//To state to user of that no comparison articles could be found 
            }

            foreach (string item in Links)
            {
                Console.WriteLine(item);
                Console.WriteLine();

            }


        }


        public void ContentsofEachC()
        {




            int NumberLink = 1;
            foreach (string item in HtmlofEachC)
            {

                Console.WriteLine("lINK NUMBER " + NumberLink);
                Intermeddiate = item;



                //Title of C



                Console.WriteLine();
                Console.WriteLine("Title may be Found");
                Console.WriteLine();
                TitlesofC();
                ProbabilityofFakeNewsC -= 40;





                if (Intermeddiate.Contains("FAKE") || Intermeddiate.Contains(" VERDICT FAKE"))
                {
                    ProbabilityofFakeNewsC -= 1000000;
                }//Incase if fact checker has been found 



                CompressionofeachC();// Intermeidate changed here dw
                CountingnumofWordsofeachC();
                // QoutesofC(); //Because does not wok properly 

                SentiForEachC();




                TitlestartPresntC = false;
                TitleendPresentC = false;

                Intermeddiate = "";
                NumberLink += 1;

            }



        }
        public void CompressionofeachC()
        {
            var irrelevantPWords = new List<string>
                    {
                        " and ", " is ", " the "," for ",
                           " first ", " on ", " because "," to ", " but "," of ",
                         " was " , " in ","<p>" , "</p>","<p" , "p>","<strong>",
                          "</strong>","~","`","!","#","$","%","^","*","_","+","=","{","}",
                          "[","]","|",":","<",">","/","&nbsp;"

                    };


            var Majorsoffakenewsdetected = new List<string> //list of words that correlate to a fake news article being detected , if the c article contians any of these more likely the article the user is reading is fake 
                    {

           "false news","fake stories","false information","untrustworthy information", "false stories",
                "fake news stories","fake news message","incorrect information",
          "fib","pork pie","propaganda","false story","hoax","phony news","paper hoax",
           "falsity of information","unreliable information","inadequate information",
           "made-up stories","doubtful information","misleading information","balderdash",
           "series of fibs","unsubstantiated information","tea","false reports","bad information",
            "fairy story","fish story","fish tale","cock-and-bull story","fallacious information",
            "pseudo event","fake","fake information","inexact information","imprecise information",
            "wrong information","erroneous information","faulty information","unsound information",
            "untrue information","not true information","not right information",
            "falsified information"

                    };







            var matches = Regex.Matches(Intermeddiate.ToString(), "<p>(.*?)</p>");

            string Words = "";

            //Removing Irrelvant text
            foreach (var m in matches)
            {


                string PureText = m.ToString();



                foreach (var irrelevantWord in irrelevantPWords)
                {
                    while (PureText.IndexOf(irrelevantWord) > -1)
                    {
                        PureText = PureText.Replace(irrelevantWord, string.Empty + " ");


                    }
                }

                PureText.ToString();
                if (PureText.Contains("<br>") && PureText.Contains("</br>"))
                {
                    PureText.Remove(Words.IndexOf("<br>"), PureText.IndexOf("</br>"));
                }//improving text

                string[] ssizes = PureText.Split(' ', '\t', '\n');
                foreach (var word in ssizes)
                {
                    string y = word.ToString();

                    var irrelevantPWords2 = new List<string>
                    {
                        "data-", "href", "span","target=", "_blank", "rel=", "class="
                        // insert irrelvant words here
                    }; //add more irrelvant words can add stuff like the p> glitch


                    foreach (string item in irrelevantPWords2)
                    {
                        if (y.Contains(item))
                        {
                            word.Replace(y, " ");

                        }


                    }

                    Words += y;
                    Words += " ";
                    Console.Write(y + " ");
                    // Words.Remove(Words.IndexOf("<br>"), Words.IndexOf("</br>"));


                    if (RelevantWordsfromB.Contains(y))
                    {


                        ProbabilityofFakeNews -= 0.6;
                        ProbabilityofBiengBiast -= 2;
                        RelevantWordsfromC.Add(y);
                    }

                    else
                    {
                        ProbabilityofFakeNewsC -= 0.3;
                        ProbabilityofBiengBiastC -= 2;
                        RelevantWordsfromC.Add(y);
                    }

                }

            }

            Console.WriteLine(Words);
            PurifedEachC.Add(Words);
            lengthofeachC.Add(Words.Length);
            Intermeddiate = Words;

            foreach (string item in Majorsoffakenewsdetected)
            {
                if (Intermeddiate.Contains(item))
                {
                    ProbabilityofFakeNewsC -= 50;
                }


            }


            ChecksafeforC();
            Words = "";


        }


        public void ChecksafeforC()
        {
            //Checks to see if article being read conians same sentencesas in the article the user is 
            //reading as some fake news articles are copy and pasted from website to website
            string[] sentencesc = Regex.Split(Intermeddiate, @"(?<=[\.!\?,])\s+");
            string[] sentencesb = Regex.Split(PureBasetext2, @"(?<=[\.!\?,])\s+");

            foreach (string sentence in sentencesc)
            {

                Console.WriteLine(sentence.ToString());

                string C = sentence.ToString();

                foreach (string item in sentencesb)
                {
                    if (item.Contains(C))
                    {
                        //Points
                        ProbabilityofFakeNewsC -= 2;

                        if (item == C)
                        {
                            ProbabilityofFakeNews -= 8;

                        }


                    }
                    else
                    {
                        //Points
                        ProbabilityofFakeNews -= 0.05;

                        ProbabilityofFakeNewsC += 0.04;



                    }

                }


            }
        } //Done inCompressionofEachC



        public void TitlesofC()
        {

            try
            {



                var matches = Regex.Matches(Intermeddiate, @"(?<=<title[\s\n]*>[\s\n]*)(.(?![\s\n]*</title[\s\n]*>))*");
                //collects title by regex
                foreach (Match item in matches)
                {
                    string intermediatetitle = item.ToString();


                    var irrelevantPWords = new List<string>
                    {
                          " and ", " is ", " the "," if "," for ", " first ", " on ", " because "," to ", " but "," of ", " was " , " in ","<p>" , "</p>","<p" , "p>", "</a>" ,"<a>","<strong>", "</strong>"
                        // insert irrelvant words here
                    }; //add more irrelvant words can add stuff like the p> glitch


                    foreach (string word in irrelevantPWords)
                    {
                        if (intermediatetitle.Contains(word))
                        {

                            intermediatetitle = intermediatetitle.Replace(word, string.Empty + " ");
                            TitlestartPresnt = true;
                            TitleendPresent = true;

                        }



                    }





                    char[] chars = { '.', ',', ';', ':', '?', '/', ' ' };  //not all titles are relevant so we determine if the title is by how many of th same words are in the titleof the article and th artile the user is reading 
                    int wordsocntianed = 0;

                    string[] Titlewords = Title.Split(chars);
                    foreach (string wordoftitle in Titlewords)
                    {
                        if (intermediatetitle.Contains(wordoftitle))
                        {
                            wordsocntianed += 1;
                            if (wordsocntianed * 1.5 > Titlewords.Count())
                            {

                                TitlesCList.Add(intermediatetitle);
                                break;

                            }

                        }

                    }
                    Console.WriteLine();


                    //Fialed implementation so that only relevant titles are used, later improved version works 
                    //foreach (string word in lol)
                    //{
                    //    if (intermediatetitle.Contains(word))
                    //    {
                    //        wordsin += 1;
                    //        if (wordsin *2 > wordtitlecount)
                    //        {
                    //            TitlestartPresntC = true;
                    //            TitleendPresentC = true;

                    //            TitlesCList.Add(intermediatetitle);
                    //            Console.WriteLine(intermediatetitle);
                    //            Console.ReadKey();
                    //            break;

                    //        }
                    //        break;



                    //    }
                    //    else
                    //    {
                    //        Console.WriteLine("This title is probably not HERE");
                    //        Console.WriteLine(intermediatetitle);
                    //    }

                    //}



                }

            }
            catch (Exception)
            {


            }

            foreach (string item in TitlesCList)
            {
                Console.WriteLine();
                Console.WriteLine(item);

            }


        }

        public void CountingnumofWordsofeachC()
        {



            //Counting Number of each word
            string Word = Intermeddiate.ToString();


            int Count = Word.Split(new char[] { ' ', '.', ',', '?' }, StringSplitOptions.RemoveEmptyEntries).Length;
            string text = Word;
            char[] chars = { ' ', '.', ',', ';', ':', '?', '\n', '\r' };
            // split words
            string[] words = text.Split(chars);

            // iterate over the word collection to count occurrences



            foreach (string word in words)
            {
                string w = word.Trim().ToLower();

                if (!statsC.ContainsKey(w))
                {
                    // add new word to collection
                    statsC.Add(w, 1);
                    TotalSamewords += 1;

                }
                else
                {
                    // update word occurrence count
                    statsC[w] += 1;
                    TotalSamewords += 1;
                }
            }




            foreach (var pair in statsC)
            {

                Console.WriteLine("Total occurrences of {0}: {1}", pair.Key, pair.Value);

                if (pair.Value > 5)
                {
                    WordsforSpellCheck.Add(pair.Key);    //adds to words that can be spell checked as exist over a certain amount of times
                }


            }
            Console.WriteLine();
            Console.WriteLine("All words for spell check");
            foreach (string item in WordsforSpellCheck)
            {
                Console.WriteLine();
                Console.WriteLine(item);
                Console.WriteLine();
            }


            Console.WriteLine("Number of words in this article = " + Count);
            LenghofCAll += Count;
            Console.WriteLine("<------------------------------Complete C Word Collection of this article -------------------------->");


        }

        public void LENGTHChecks()
        {




            try
            {
                if (TotalNumofWordsofB + 1000 >= LenghofCAll / URLCount || TotalNumofWordsofB + 1000 <= LenghofCAll / URLCount)
                {
                    Console.WriteLine("Suspciously high or low  word count");

                    ProbabilityofFakeNews += 50;


                    foreach (int item in lengthofeachC)
                    {
                        if (TotalNumofWordsofB + 50 >= item || TotalNumofWordsofB + 50 <= item)
                        {
                            Console.WriteLine("Suspciously close word count ");

                            ProbabilityofFakeNews += 100;
                        }

                    }


                }
                else
                {
                    //decent amount of word count differnetiation
                    ProbabilityofFakeNews -= 100;

                }





                Console.WriteLine("Total Number of neccesary Words Collected from all  HTMl " + LenghofCAll);
                Console.WriteLine("Average  Number of neccesary Words Collected from all  HTMl " + LenghofCAll / URLCount);
                Console.WriteLine("Total Number of words collected from the user article  " + TotalNumofWordsofB);

            }
            catch (Exception)
            {


            }



        }


        public void QoutesofC()
        {
            Console.WriteLine("Qoutes");  //extract qoutes for comparison

            string b = Intermeddiate.ToString();

            string beep = Intermeddiate.Replace("\"", "\"");
            string beep2 = beep.Replace("&#", "\"") + 4;

            //WACKAWACKA
            MatchCollection Qoutes = Regex.Matches(beep2, "\"([^\"]*)\"");  //   (["'&#])(?:(?=(\\?))\2.)*?\1


            foreach (Match c in Qoutes)
            {

                string a = c.ToString();


                if (!QouteListC.Contains(a))
                {


                    //Only adds to qoutesofc if qoute not already found so not to many qoutes 

                    QouteListC.Add(a);
                    NumofQOutesC += 1;
                    Console.WriteLine(a);


                }
                else
                {
                    Console.WriteLine("This qoute already in list  ______>    " + a);
                    ProbabilityofBiengBiastC -= 0.1;
                    ProbabilityofFakeNewsC -= 0.2;

                }

                if (a.Contains("body - link") || a.Contains("body-link") || a.Contains("www") || a.Contains("BL-") || a.Contains("href") || a.Contains("span") || a.Contains("target=") || a.Contains("_blank") || a.Contains("rel=") || a.Contains("class=") || a.Contains("href"))
                {
                    QouteListC.Remove(a);

                }




            }


            if (NumofQOutesC / URLCount < 5)
            {
                ProbabilityofFakeNewsC += 50;
                ProbabilityofBiengBiastC += 50;
            }

            foreach (string item in QouteListC)
            {
                Console.WriteLine(item);
                Console.WriteLine();
                Console.WriteLine();

            }

            Console.WriteLine(" Total Number of Qoutes in this Comparing Article = " + QouteListC.Count);

        }

        public void ComparingQoutes()
        {//QoutesofB   //QouteListC
            //Does not run in the final program 
            try
            {
                if (NumofQoutesB > 5 && NumofQOutesC / URLCount > 5)
                {


                    if (NumofQoutesB > NumofQOutesC / URLCount)
                    {
                        ProbabilityofBiengBiast += 20;
                        ProbabilityofFakeNews += 50;
                    }
                    else
                    {
                        ProbabilityofBiengBiast -= 20;
                        ProbabilityofFakeNews -= 50;

                        ProbabilityofBiengBiastC -= 60;
                        ProbabilityofFakeNewsC -= 30;

                    }


                    foreach (string C in QouteListC)
                    {

                        foreach (string B in QoutesofB)
                        {

                            if (C.Contains(B))
                            {


                                ProbabilityofBiengBiast -= 0.05;
                                ProbabilityofFakeNews -= 0.09;

                                //Exact same QOute
                                if (C == B)
                                {
                                    ProbabilityofBiengBiast += 0.01;
                                    ProbabilityofFakeNews += 0.01;

                                    ProbabilityofBiengBiastC += 2;
                                    ProbabilityofFakeNewsC += 2;

                                }


                            }
                            else
                            {
                                //Suspicious
                                Console.WriteLine("This qoute not found in any of comparing articles");
                                //Points
                                ProbabilityofBiengBiast += 0.9;
                                ProbabilityofFakeNews += 0.9;


                            }
                        }
                    }


                }
                else
                {
                    Console.WriteLine("Considerably low amount of Qoutes, removing points");
                    ProbabilityofBiengBiast += 50;
                    ProbabilityofFakeNews += 50;
                }

            }
            catch (Exception ex)
            {

                Console.WriteLine(ex);
                Console.WriteLine("Most likely error no articles could be found");

            }







        }

        public void ComparingNumberofUseofWords()
        {
            try
            {


                foreach (string item in statsB.Keys)
                {
                    if (statsC.ContainsKey(item))
                    {


                        int value1;
                        int value2;
                        statsB.TryGetValue(item, out value1);     //extracts the key.value of stats b and stats c
                        statsC.TryGetValue(item, out value2);     //and stats c



                        if (value1 > value2 / URLCount)    // compares them
                        {
                            //Points
                            ProbabilityofBiengBiast += 0.09;
                            Console.WriteLine("LOL");

                            //         Console.WriteLine("A");
                        }
                        else if (value1 < value2 / URLCount)
                        {
                            //Points
                            ProbabilityofBiengBiast += 0.09;
                            Console.WriteLine("OLO");

                            //           Console.WriteLine("B");
                        }
                        else
                        {
                            //Points

                            ProbabilityofBiengBiast -= 0.9;
                            Console.WriteLine("QWQ");

                            //            Console.WriteLine("C");
                        }


                        //}


                    }

                }

            }
            catch (Exception)
            {
                Console.WriteLine("An error occured");
                Console.WriteLine("Most likely no links could be found");

            }

        }

        public void SentiInput()
        {
            //SentiInputSentences

            char[] chars = { '.', ',', ';', ':', '?', '/' };


            string[] words = Intermeddiate.Split(chars);  //splits the  data the user is reading into sentences  


            foreach (string match in words)
            {
                if (!match.Contains("<a") || !match.Contains("a>") || !match.Contains("<b") || !match.Contains("b>") || match.Contains("a") || match.Contains("e") || match.Contains("i") || match.Contains("o") || match.Contains("u") || match.Contains("a"))
                {
                    SentiInputSentences.Add(match.ToString());
                    Console.WriteLine(match);

                }



            }




        }
        public void SentiForEachC()
        {


            MLContext mlContext = new MLContext();//Setup for C
            TrainTestData splitDataView = LoadData(mlContext);
            ITransformer model = BuildAndTrainModel(mlContext, splitDataView.TrainSet);   //trianing and evaluation of senti
            Evaluate(mlContext, model, splitDataView.TestSet);
            UseModelWithSingleItem(mlContext, model);
            SentiInputSentences.Clear();   //clears data from the artile the user is reading


            char[] chars = { '.', ',', ';', ':', '?' };


            string[] words = Intermeddiate.Split(chars);


            foreach (string match in words)
            {
                if (!match.Contains("<a") || !match.Contains("a>") || !match.Contains("<b") || !match.Contains("b>") || match.Contains("a") || match.Contains("e") || match.Contains("i") || match.Contains("o") || match.Contains("u") || match.Contains("a"))
                {
                    SentiInputSentences.Add(match.ToString());

                }



            }

            UseModelWithBatchItemsC(mlContext, model, SentiInputSentences);
            Console.WriteLine();


            if (TitlestartPresnt && TitleendPresent && TitlestartPresnt && TitleendPresent)  //only run title of c senti if title of article user is reading is found
            {

                UseModelWithBatchItemsCTitle(mlContext, model, SentiInputSentences);

            }






        }


        public void UseModelWithBatchItemsC(MLContext mlContext, ITransformer model, List<string> SentiInputSentences)
        {
            int Positive = 0;
            int Negative = 0;
            foreach (string item in SentiInputSentences)
            {


                string sentence = item.ToString();

                IEnumerable<SentimentData> sentiments = new[]
                {
             
                //new SentimentData
                //{
                //   SentimentText = "I love this spaghetti."
                //},
               new SentimentData
                {
                   SentimentText = sentence
                }

               };


                IDataView batchComments = mlContext.Data.LoadFromEnumerable(sentiments);

                IDataView predictions = model.Transform(batchComments);

                // Use model to predict whether comment data is Positive (1) or Negative (0).
                IEnumerable<SentimentPrediction> predictedResults = mlContext.Data.CreateEnumerable<SentimentPrediction>(predictions, reuseRowObject: false);

                Console.WriteLine();

                Console.WriteLine("=============== Prediction Test of loaded model with multiple samples ===============");

                foreach (SentimentPrediction prediction in predictedResults)
                {
                    Console.WriteLine($"Sentiment: {prediction.SentimentText} | Prediction: {(Convert.ToBoolean(prediction.Prediction) ? "Positive" : "Negative")} | Probability: {prediction.Probability} ");
                    if (prediction.Probability <= 0.5)
                    {
                        Positive += 1;
                    }
                    else
                    {
                        Negative += 1;
                    }
                }
            }
            Console.WriteLine("=============== End of predictions ===============");
            Console.WriteLine("Num of Positives calculated " + Positive);
            Console.WriteLine("Num of Negatives calculated " + Negative);

            totalPositiveC += Positive;
            totalNegativeC += Negative;

            totalPositiveCAVG += Positive / URLCount;
            totalNegativeCAVG += Negative / URLCount;

            Positive = 0;
            Negative = 0;


        } //ForCText

        public void UseModelWithBatchItemsCTitle(MLContext mlContext, ITransformer model, List<string> SentiInputSentences)
        {


            int PositiveT = 0;
            int NegativeT = 0;
            foreach (var item in TitlesCList)
            {


                string sentence = item;

                IEnumerable<SentimentData> sentiments = new[]
                {
             
                //new SentimentData
                //{
                //   SentimentText = "I love this spaghetti."
                //},
               new SentimentData
                {
                   SentimentText = sentence
                }

               };


                IDataView batchComments = mlContext.Data.LoadFromEnumerable(sentiments);

                IDataView predictions = model.Transform(batchComments);

                // Use model to predict whether comment data is Positive (1) or Negative (0).
                IEnumerable<SentimentPrediction> predictedResults = mlContext.Data.CreateEnumerable<SentimentPrediction>(predictions, reuseRowObject: false);

                Console.WriteLine();

                Console.WriteLine("=============== Prediction Test of loaded model with multiple samples ===============");

                foreach (SentimentPrediction prediction in predictedResults)
                {
                    Console.WriteLine($"Sentiment: {prediction.SentimentText} | Prediction: {(Convert.ToBoolean(prediction.Prediction) ? "Positive" : "Negative")} | Probability: {prediction.Probability} ");
                    if (prediction.Probability <= 0.5)
                    {
                        PositiveT += 1;
                    }
                    else
                    {
                        NegativeT += 1;
                    }
                }

                Console.WriteLine("=============== End of predictions ===============");






            }
            PositiveTitles += PositiveT;
            NegativeTitles += NegativeT;
            Console.WriteLine("Total Positive Titles found =  " + PositiveTitles);//Of comparison articles 
            Console.WriteLine("Total Negatives Titles found =  " + NegativeTitles);//Of comparison articles 


            PositiveT = 0;
            NegativeT = 0;




        }  //ForCTitles //Checks if Fake 
        public void UseModelWithBatchItemsBTitle(MLContext mlContext, ITransformer model, List<string> SentiInputSentences)//ForBTitle
        {


            int PositiveTB = 0;
            int NegativeTB = 0;



            string sentence = Title;

            IEnumerable<SentimentData> sentiments = new[]
            {
             
                //new SentimentData
                //{
                //   SentimentText = "I love this spaghetti."
                //},
               new SentimentData
                {
                   SentimentText = sentence
                }

               };


            IDataView batchComments = mlContext.Data.LoadFromEnumerable(sentiments);

            IDataView predictions = model.Transform(batchComments);

            // Use model to predict whether comment data is Positive (1) or Negative (0).
            IEnumerable<SentimentPrediction> predictedResults = mlContext.Data.CreateEnumerable<SentimentPrediction>(predictions, reuseRowObject: false);

            Console.WriteLine();

            Console.WriteLine("=============== Prediction Test of loaded model with multiple samples ===============");

            foreach (SentimentPrediction prediction in predictedResults)
            {
                Console.WriteLine($"Sentiment: {prediction.SentimentText} | Prediction: {(Convert.ToBoolean(prediction.Prediction) ? "Positive" : "Negative")} | Probability: {prediction.Probability} ");
                if (prediction.Probability <= 0.5)
                {
                    PositiveTB += 1;
                }
                else
                {
                    NegativeTB += 1;
                }
                PositiveTitleB = PositiveTB;
                NegaitiveTitleB = NegativeTB;

            }

            Console.WriteLine("=============== End of predictions ===============");





        }



        public void CompareTitle()
        {

            if (TitlestartPresnt && TitleendPresent)
            {
                if (PositiveTitleB > NegaitiveTitleB)   //sorting of if title of being read article os postive or negative , either will be 1 or 0 
                {
                    BasePosOrneg = true;   //true represents psotive
                }
                else
                {
                    BasePosOrneg = false; //false represents negative
                }


                if (PositiveTitles > NegativeTitles)
                {
                    CPosOrNeg = true;    //same done for comparing articles but both may be more that 1 or 0
                }
                else
                {
                    CPosOrNeg = false;
                }

                Console.WriteLine("<-----------------------------SENTIResults--------------------------->");


                if (BasePosOrneg)
                {
                    Console.WriteLine();
                    Console.WriteLine("Article TITLE considered Positive");
                    Console.WriteLine();
                    ////Points
                    //ProbabilityofFakeNews -= 50;
                    //ProbabilityofBiengBiast -= 60;

                }
                else
                {

                    Console.WriteLine("Article title considered Negative");

                }
                if (CPosOrNeg)
                {

                    Console.WriteLine("Articles titles  considered Positive");


                }
                else
                {
                    Console.WriteLine("Articles  titles considered Negative");
                }

                if (!BasePosOrneg && CPosOrNeg || BasePosOrneg && !CPosOrNeg)      //placing points
                {


                    Console.WriteLine("Article titles considered Oppsite to other articles");

                    //Points

                    ProbabilityofBiengBiast += 120;
                    ProbabilityofFakeNews += 50;

                    ProbabilityofBiengBiastC -= 120;
                    ProbabilityofFakeNewsC -= 50;

                }
                if (BasePosOrneg && CPosOrNeg || !BasePosOrneg && !CPosOrNeg)
                {

                    Console.WriteLine(" Both the article and the comparing articles are considered the same");


                    ProbabilityofBiengBiast -= 120;
                    ProbabilityofFakeNews -= 50;





                }


                char[] chars = { '.', ',', ';', ':', '?', ' ' };
                string[] Titlewords = Title.Split(chars);

                foreach (string item in TitlesCList)
                {
                    string[] TitlewordsofC = item.Split(chars);

                    foreach (string CTitleword in TitlewordsofC)
                    {
                        foreach (var BTITLEWRD in Titlewords)
                        {
                            if (CTitleword.Contains(BTITLEWRD))
                            {
                                ProbabilityofFakeNews -= 2;

                            }
                            else
                            {
                                ProbabilityofFakeNews += 2;

                            }

                        }

                    }

                }




                //ProbabilityofFakeNews += 400;
                //ProbabilityofBiengBiast += 300;


                //ProbabilityofBiengBiastC += 600;
                //ProbabilityofFakeNewsC += 800;
                //AuthenticityC -= 800;

            }





        }



        public void ComparingArticles()
        {

            //Preperation
            bool TrueFalseB = false;
            bool TrueFalseC = false;
            //For C
            if (totalPositiveCAVG > totalNegativeCAVG)  //sorts articles into if postive or negative connotation
            {
                TrueFalseC = true;
            }
            if (totalNegativeCAVG > totalPositiveCAVG)
            {
                TrueFalseC = false;
            }

            //Forb

            if (totalPositive > totalNegative)
            {
                TrueFalseB = true;
            }
            if (totalNegative > totalPositive)
            {
                TrueFalseB = false;  //Predicted to be false  
            }


            //Comparing
            if (TrueFalseC && TrueFalseB || !TrueFalseC && !TrueFalseB)
            {
                //Calculated Positive article 
                //points
                ProbabilityofFakeNews -= 200;
                ProbabilityofBiengBiast -= 200;



            }

            else
            {
                ProbabilityofFakeNews += 200;
                ProbabilityofBiengBiast += 200;

            }



            //Comparing article to title
            if (BasePosOrneg && !TrueFalseB || !BasePosOrneg && TrueFalseB)
            {
                //Title and Article considered opposite
                //Points
                ProbabilityofFakeNews += 100;


            }
            else
            {
                //Points
                ProbabilityofFakeNews -= 100;


            }


            foreach (string C in RelevantWordsfromC)
            {
                if (RelevantWordsfromB.Contains(C))
                {
                    ProbabilityofFakeNews -= 0.9;
                    ProbabilityofBiengBiast -= 0.9;
                    ProbabilityofFakeNewsC += 0.5;
                    ProbabilityofBiengBiastC += 0.5;
                }
                else
                {
                    ProbabilityofFakeNews += 0.5;
                    ProbabilityofBiengBiast += 0.5;
                    ProbabilityofFakeNewsC += 0.5;
                    ProbabilityofBiengBiastC += 0.5;
                }



            }









        }



        public void PointsOutput()
        {



            Console.WriteLine();
            Console.WriteLine();
            Console.WriteLine();
            Console.WriteLine();

            Console.BackgroundColor = ConsoleColor.Blue;
            Console.WriteLine(Title);
            Console.WriteLine();
            Console.WriteLine();






            Console.WriteLine("<--------------------------------------------------------------------------->");
            Console.WriteLine("<--------------------------------------------------------------------------->");
            Console.WriteLine("<--------------------------------------------------------------------------->");
            Console.WriteLine("<--------------------------------------------------------------------------->");
            Console.WriteLine("<--------------------------------------------------------------------------->");
            Console.WriteLine("<-------------------------------REUSLTS Ready------------------------------------->");
            Console.WriteLine();
            Console.WriteLine();
            Console.WriteLine(Title);
            Console.WriteLine("The article you were reading was compared to  " + URLCount + " other articles on the internet ");
            Console.WriteLine("PLEASE CONSIDER LINKS STATED BELOW THAT THE ARTICLE YOU WERE READING WAS COMPARED TO AS SMALL CHANCE THEM MAY NOT BE RELATED");
            Console.WriteLine();
            Console.WriteLine();

            Console.WriteLine("Found these to compare to");

            foreach (string item in Links)
            {

                Console.WriteLine(item);
                Console.WriteLine();
            }


            if (URLCountcheck)
            {
                Console.BackgroundColor = ConsoleColor.Red;
                Console.WriteLine("Others were hard to source");
                Console.WriteLine("Not valid Results no Comparison URs could be found ");
                Console.WriteLine("Only some checks were completed");
                Console.BackgroundColor = ConsoleColor.Black;


            }

            Console.WriteLine();
            Console.WriteLine();
            Console.WriteLine("Used this as the websearch  " + urlAddressC);



            if (BasePosOrneg)
            {
                Console.WriteLine();
                Console.WriteLine("Article TITLE considered Positive");
                Console.WriteLine();

            }
            else
            {

                Console.WriteLine("Article title considered Negative");

            }
            if (CPosOrNeg)
            {

                Console.WriteLine("Articles titles  considered Positive");
            }
            else
            {
                Console.WriteLine("Articles  titles considered Negative");
            }

            if (!BasePosOrneg && CPosOrNeg || BasePosOrneg && !CPosOrNeg)
            {

                Console.WriteLine("Article titles considered Oppsite to other articles");

            }
            if (BasePosOrneg && CPosOrNeg || !BasePosOrneg && !CPosOrNeg)
            {

                Console.WriteLine(" Both the article and the comparing articles are considered the same");
            }


            Console.WriteLine();
            Console.WriteLine();

            Console.WriteLine("Total Number of neccesary Words Collected from all  HTMl " + LenghofCAll);
            Console.WriteLine("Average  Number of neccesary Words Collected from all  HTMl " + LenghofCAll / URLCount);
            Console.WriteLine("Total Number of words collected from the user article  " + TotalNumofWordsofB);
            Console.WriteLine();
            Console.WriteLine();
            //Protocol bias reverse mechanisim

            bool activate = false;
            if (ProbabilityofBiengBiast < 0 && ProbabilityofBiengBiastC / URLCount > 0 || ProbabilityofBiengBiast > 0 && ProbabilityofBiengBiastC / URLCount < 0)
            {
                activate = true;

            }




            //Fake News
            if (ProbabilityofFakeNews > ProbabilityofFakeNewsC / URLCount)
            {

                if (ProbabilityofFakeNews - 100 > ProbabilityofFakeNewsC / URLCount)
                {
                    //Deffo fake
                    Console.BackgroundColor = ConsoleColor.Red;
                    //  Console.WriteLine("The Program found it difficult to gain the correct answer so the statement below is correct , ignore the instructions next to the numbers");
                    if (activate)
                    {
                        Console.WriteLine("Porgram detected the comparison articles may be not affective, consider results");
                        Console.WriteLine("The article you are reading is most likely fake");
                        Console.WriteLine();
                        Console.WriteLine("Probabilit of fake news count = " + ProbabilityofFakeNews + "/" + ProbabilityofFakeNewsC / URLCount + "<----------if above is fake , if below is real ");
                        Console.WriteLine();
                        RoundsFakeNews.Add("The article you are reading is most likely fake");

                    }
                    else
                    {
                        Console.WriteLine("The article you are reading is most likely fake");
                        Console.WriteLine();
                        Console.WriteLine("Probabilit of fake news count = " + ProbabilityofFakeNews + "/" + ProbabilityofFakeNewsC / URLCount + "<----------if above is fake , if below is real ");
                        Console.WriteLine();
                        RoundsFakeNews.Add("The article you are reading is most likely fake");

                    }



                }
                else
                {
                    //Flip affect as may be true


                    if (activate)
                    {
                        Console.WriteLine("Porgram detected the comparison articles may be not affective, consider results and articles compared to (scroll up/down)");
                        Console.BackgroundColor = ConsoleColor.DarkGreen;
                        Console.WriteLine("The Program found it difficult to gain the correct answer so the statement below is correct , ignore the instructions next to the numbers");
                        Console.WriteLine("The article you are reading is most likely truthful");
                        Console.WriteLine();
                        Console.WriteLine("Probability of fake news count = " + ProbabilityofFakeNews + "/" + ProbabilityofFakeNewsC / URLCount + "<----------if above is fake , if below is real ");
                        Console.WriteLine();
                        Console.WriteLine();
                        Console.WriteLine();
                        RoundsFakeNews.Add("The article you are reading is most likely truthful");

                    }
                    else
                    {
                        Console.BackgroundColor = ConsoleColor.DarkGreen;
                        Console.WriteLine("The Program found it difficult to gain the correct answer so the statement below is correct , ignore the instructions next to the numbers");
                        Console.WriteLine("The article you are reading is most likely truthful");
                        Console.WriteLine();
                        Console.WriteLine("Probability of fake news count = " + ProbabilityofFakeNews + "/" + ProbabilityofFakeNewsC / URLCount + "<----------if above is fake , if below is real ");
                        Console.WriteLine();
                        Console.WriteLine();
                        Console.WriteLine();
                        RoundsFakeNews.Add("The article you are reading is most likely truthful");

                    }


                }

            }
            if (ProbabilityofFakeNews < ProbabilityofFakeNewsC / URLCount)
            {
                if (activate)
                {
                    Console.WriteLine("Porgram detected the comparison articles may be not affective, consider result");
                    Console.BackgroundColor = ConsoleColor.DarkGreen;
                    Console.WriteLine("The article you are reading is most likely truthful");
                    Console.WriteLine();
                    Console.WriteLine("Probabilit of fake news count = " + ProbabilityofFakeNews + "/" + ProbabilityofFakeNewsC / URLCount + "<----------if above is fake , if below is real ");
                    Console.WriteLine();
                    Console.WriteLine();
                    Console.WriteLine();
                    RoundsFakeNews.Add("The article you are reading is most likely truthful");

                }
                else
                {
                    Console.BackgroundColor = ConsoleColor.DarkGreen;
                    Console.WriteLine("The article you are reading is most likely truthful");
                    Console.WriteLine();
                    Console.WriteLine("Probabilit of fake news count = " + ProbabilityofFakeNews + "/" + ProbabilityofFakeNewsC / URLCount + "<----------if above is fake , if below is real ");
                    Console.WriteLine();
                    Console.WriteLine();
                    Console.WriteLine();
                    RoundsFakeNews.Add("The article you are reading is most likely truthful");

                }


            }



            //Bias
            if (ProbabilityofBiengBiast < ProbabilityofBiengBiastC / URLCount)
            {
                Console.WriteLine("Cant really give a definitive answer for if a article is biast or not as always up for debate so the two numbers below should be as close togehter as possible");
                Console.WriteLine();
                Console.WriteLine("Probabilit of being biast count = " + ProbabilityofBiengBiast + "/" + ProbabilityofBiengBiastC / URLCount + "    Closer together , less biast. Further apart , More likely biast");
                Console.WriteLine();
                RoundsBias.Add("Probabilit of being biast count = " + ProbabilityofBiengBiast + " / " + ProbabilityofBiengBiastC / URLCount);

            }
            if (ProbabilityofBiengBiast > ProbabilityofBiengBiastC / URLCount)
            {
                Console.WriteLine("Cant really give a definitive answer for if a article is biast or not as always up for debate so the two numbers below should be as close togehter as possible");
                Console.WriteLine();
                Console.WriteLine("Probabilit of being biast count = " + ProbabilityofBiengBiast + "/" + ProbabilityofBiengBiastC / URLCount + "    Closer together , less biast. Further apart , More likely biast");
                Console.WriteLine();
                RoundsBias.Add("Probabilit of being biast count = " + ProbabilityofBiengBiast + " / " + ProbabilityofBiengBiastC / URLCount);

            }

            Console.WriteLine();
            Console.WriteLine();
            Console.WriteLine();
            Console.WriteLine("<------------------------------Finals------------------------------------------->");

            foreach (string item in RoundsFakeNews)
            {
                Console.WriteLine(item);

            }
            foreach (string item in RoundsBias)
            {
                Console.WriteLine(item);

            }
            Console.BackgroundColor = ConsoleColor.Black;

            Console.WriteLine();

            if (Console.ReadLine() == "321")
            {
                Console.WriteLine();
                Console.WriteLine();
                foreach (string item in TriedURLS)
                {

                    Console.WriteLine();
                    Console.WriteLine(item);
                    Console.WriteLine();
                    Console.WriteLine();

                }

            }
            if (Console.ReadLine() == "123")
            {
                Console.WriteLine();
                Console.WriteLine();
                foreach (string item in TitlesCList)
                {

                    Console.WriteLine();
                    Console.WriteLine(item);
                    Console.WriteLine();
                    Console.WriteLine();

                }

            }


        }

        public void PointsOutputforTest()
        {

            if (Delta == false)
            {
                Console.WriteLine("Others were hard to source");

            }


            if (ProbabilityofFakeNews > ProbabilityofFakeNewsC / URLCount)
            {
                if (ProbabilityofFakeNews - 100 > ProbabilityofFakeNewsC / URLCount)
                {
                    Console.WriteLine("The article you are reading is most likely fake");
                    Console.WriteLine();
                    Console.WriteLine("Probability of fake news count = " + ProbabilityofFakeNews + "/" + ProbabilityofFakeNewsC / URLCount + "<----------if above is fake , if below is real ");
                    Console.WriteLine();


                    if (expectedresult == "false" || expectedresult == "FALSE" || expectedresult == "False")
                    {
                        //   Console.BackgroundColor = ConsoleColor.Green;
                        Console.WriteLine(urlAddress);
                        Console.WriteLine(Title);
                        GotRight += 1;

                        // Corrects.Add(urlAddress);
                        string qwq = (ProbabilityofFakeNews + " / " + ProbabilityofFakeNewsC / URLCount).ToString();
                        Corrects.Add(Title + "   " + qwq + urlAddressC);



                        Console.WriteLine("GOT RIGHT COUNT =" + GotRight);
                        Console.WriteLine("Got this one right");
                        //  Console.BackgroundColor = ConsoleColor.Black;
                    }



                    if (expectedresult == "True" || expectedresult == "TRUE" || expectedresult == "true")
                    {
                        //    Console.BackgroundColor = ConsoleColor.Red;
                        Console.WriteLine(urlAddress);
                        Console.WriteLine(Title);

                        GotWrong += 1;

                        // Incorrects.Add(urlAddress);
                        string qwq = (ProbabilityofFakeNews + " / " + ProbabilityofFakeNewsC / URLCount).ToString();
                        Incorrects.Add(Title + "     " + qwq + "   " + urlAddressC);


                        Console.WriteLine("GOT wrong COUNT = " + GotWrong);
                        Console.WriteLine("Got this one wrong");
                        //  Console.BackgroundColor = ConsoleColor.Black;

                    }



                }
                else
                {
                    // flip affect
                    Console.WriteLine("The Program found it difficult to gain the correct answer so the statement below is correct , ignore the instructions next to the numbers");
                    Console.WriteLine("The article you are reading is most likely True");
                    Console.WriteLine();
                    Console.WriteLine("Probability of fake news count = " + ProbabilityofFakeNews + "/" + ProbabilityofFakeNewsC / URLCount + "<----------if above is fake , if below is real ");
                    Console.WriteLine();

                    if (expectedresult == "True" || expectedresult == "TRUE" || expectedresult == "true")
                    {
                        //   Console.BackgroundColor = ConsoleColor.Green;
                        Console.WriteLine(urlAddress);
                        Console.WriteLine(Title);
                        GotRight += 1;

                        // Corrects.Add(urlAddress);
                        string qwe = (ProbabilityofFakeNews + " / " + ProbabilityofFakeNewsC / URLCount).ToString();
                        Corrects.Add(Title + "      " + qwe + urlAddressC);

                        Console.WriteLine("GOT RIGHT COUNT = " + GotRight);
                        Console.WriteLine("Got this one Right");
                        //  Console.BackgroundColor = ConsoleColor.Black;

                    }

                    if (expectedresult == "fake" || expectedresult == "Fake" || expectedresult == "FAKE")
                    {
                        // Console.BackgroundColor = ConsoleColor.Red;
                        Console.WriteLine(urlAddress);
                        Console.WriteLine(Title);
                        GotWrong += 1;

                        // Incorrects.Add(urlAddress);
                        string qwq = (ProbabilityofFakeNews + " / " + ProbabilityofFakeNewsC / URLCount).ToString();
                        Incorrects.Add(Title + "     " + qwq + urlAddressC);

                        Console.WriteLine("GOT wrong COUNT = " + GotWrong);
                        Console.WriteLine("Got this one wrong");
                        //   Console.BackgroundColor = ConsoleColor.Black;
                    }

                }





            }




            if (ProbabilityofFakeNews < ProbabilityofFakeNewsC / URLCount)
            {

                Console.WriteLine("The article you are reading is most likely truthful");
                Console.WriteLine();
                Console.WriteLine("Probabilit of fake news count = " + ProbabilityofFakeNews + "/" + ProbabilityofFakeNewsC / URLCount + "<----------if above is fake , if below is real ");
                Console.WriteLine();
                Console.WriteLine();
                Console.WriteLine();

                if (expectedresult == "fake" || expectedresult == "Fake" || expectedresult == "FAKE")
                {
                    // Console.BackgroundColor = ConsoleColor.Red;
                    Console.WriteLine(urlAddress);
                    Console.WriteLine(Title);
                    GotWrong += 1;

                    // Incorrects.Add(urlAddress);
                    string qwq = (ProbabilityofFakeNews + " / " + ProbabilityofFakeNewsC / URLCount).ToString();
                    Incorrects.Add(Title + "     " + qwq + urlAddressC);

                    Console.WriteLine("GOT wrong COUNT = " + GotWrong);
                    Console.WriteLine("Got this one wrong");
                    //   Console.BackgroundColor = ConsoleColor.Black;
                }
                if (expectedresult == "True" || expectedresult == "TRUE" || expectedresult == "true")
                {
                    //   Console.BackgroundColor = ConsoleColor.Green;
                    Console.WriteLine(urlAddress);
                    Console.WriteLine(Title);
                    GotRight += 1;

                    // Corrects.Add(urlAddress);
                    string qwe = (ProbabilityofFakeNews + " / " + ProbabilityofFakeNewsC / URLCount).ToString();
                    Corrects.Add(Title + "      " + qwe + urlAddressC);

                    Console.WriteLine("GOT RIGHT COUNT = " + GotRight);
                    Console.WriteLine("Got this one Right");
                    //  Console.BackgroundColor = ConsoleColor.Black;

                }

            }


            Console.WriteLine();
            Console.WriteLine();
            Console.WriteLine();
            Console.WriteLine();
            Console.WriteLine("<!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!>");
            Console.WriteLine("Got these ones Correct");
            Console.WriteLine("<!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!>");
            foreach (string item in Corrects)
            {
                Console.WriteLine(item);
                Console.WriteLine();

            }
            Console.WriteLine();
            Console.WriteLine();
            Console.WriteLine();
            Console.WriteLine("<!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!>");
            Console.WriteLine("Got these ones Incorrect");
            Console.WriteLine("<!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!>");
            foreach (string item in Incorrects)
            {
                Console.WriteLine(item);
                Console.WriteLine();


            }
            Console.WriteLine();
            Console.WriteLine();
            Console.WriteLine();
            Console.WriteLine("<!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!>");
            Console.WriteLine("Unknowns");
            Console.WriteLine("<!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!>");
            foreach (string item in Unknowns)
            {
                Console.WriteLine(item);
                Console.WriteLine();


            }












        }

        public void Percents()
        {
            Console.WriteLine();
            Console.WriteLine();
            Console.WriteLine();
            Console.WriteLine();
            Console.WriteLine("<-------------------------------------------Percentages-------------------------------------->");
            Console.WriteLine();
            Console.WriteLine();
            Console.WriteLine("Testing done on " + validurl.Count() + "Websites");



            double PercentageCorrect = GotRight / Beta * 100;
            double PercentageInCorrect = GotWrong / Beta * 100;

            //  Console.WriteLine("Got " + PercentageCorrect + "%  Correct and  got " + PercentageInCorrect + "%  Incorrect ");


            Console.WriteLine("Got " + GotRight + "/" + Beta + "  Correct and  got " + GotWrong + "/" + Beta + "  Incorrect and could not solve " + Unknowns.Count() + " due to reasons");
            Console.WriteLine();
            Console.WriteLine("Got " + PercentageCorrect + "%  Correct and  got " + PercentageInCorrect + "%  Incorrect and could not solve " + Unknowns.Count() + " due to reasons");
            Console.WriteLine();






        }

    }

    public class SentimentData
    {
        [LoadColumn(0)]
        public string SentimentText;

        [LoadColumn(1), ColumnName("Label")]
        public bool Sentiment;
    }

    public class SentimentPrediction : SentimentData
    {

        [ColumnName("PredictedLabel")]
        public bool Prediction { get; set; }

        public float Probability { get; set; }

        public float Score { get; set; }
    }




}







