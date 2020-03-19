# Infy

Front end src

import React, { Component } from "react";
import axios from "axios"
import { CSVLink } from "react-csv";
import _ from "underscore";
import xlsx from "xlsx";
import "bootstrap/dist/css/bootstrap.min.css";
class FileRead extends Component {
    constructor(props) {
        super(props);
        this.state = {
            jdata: "",
            groupedData: "",
            projectData: "",
            sizeOfGroup: "",
            isValid: false,
        };
    }
    randomData = data => {
        this.setState({ projectData: data });
        let d = this.state.projectData;
        var perChunk = parseInt(this.state.sizeOfGroup);
        var partners = _.chunk(_.shuffle(d), perChunk);
        var result = partners.map((f, i) => {
            let p = f.map(k => {
                k.ID = i + 1;
                return k;
            });
            return p;
        });
        this.setState({ jdata: result });
    };
    sortData = () => {
        let data = this.state.jdata;
        let arr = [].concat(...data);
        arr.sort((a, b) => {
            return a.Id - b.Id;
        });
        this.setState({ groupedData: arr, isValid: true });
    };
    handleSubmit =(event)=>{
      event.preventDefault()
      this.sortData()
    }
    gitAuto =()=>{
      const data=this.state.jdata.length
      console.log("inGitAuto");
      
      axios.get("http://localhost:2500/createrepo/"+data)
      .then(response =>{
        console.log("response");
        
      })
      .catch(err =>{
        console.log("error");
        
      })
    }
    handleChange = e => {
        e.preventDefault();
        var fileReader;
        var dataParse = "";
        var files = e.target.files,
            f = files[0];
        fileReader = new FileReader();
        fileReader.onload = e => {
            var data = e.target.result;
            let readedData = xlsx.read(data, { type: "binary" });
            const wsname = readedData.SheetNames[0];
            const ws = readedData.Sheets[wsname];
            /* Convert array to json*/
            dataParse = xlsx.utils.sheet_to_json(ws, { header: 2 });
            this.randomData(dataParse);
        };
        fileReader.readAsBinaryString(f);
    };
    handleClick = e => {
        e.preventDefault();
        this.setState({ sizeOfGroup: e.target.value });
    };
    render = () => {
        return (
            <div className="Register">
                <div className="row">
                    <div className="col-md-4 offset-md-4">
                        <br />
                        <div className="card">
                            <div className="card-header">
                                <h3>Project Group</h3>
                            </div>
                            <div className="card-body">
                                <form onSubmit={this.handleSubmit}>
                                    <div className="form-group">
                                        <label htmlFor="size">Size</label>
                                        <input type="number" name="size" id="size" onChange={this.handleClick} />
                                    </div>
                                    <div className="form-group">
                                        <input type="file" id="file" className="input-file" onChange={this.handleChange} />
                                    </div>
                                    <button type="submit" name="register" className="btn btn-primary">Create</button>
                                    <div className="form-group">
                                        {this.state.groupedData != "" ? (
                                            <React.Fragment>
                                                <div className="form-control">
                                                    <CSVLink data={this.state.groupedData}>Download me</CSVLink>
                                                </div>
                                            </React.Fragment>
                                        ) : null}
                                    </div> 
                                    <button  name="CreateRepository" className="btn btn-primary"onClick={this.gitAuto}>CreateRepository</button>
                                </form>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        );
    };
}
export default FileRead;





import React, { Component, Fragment } from "react";


class Bahenchod extends Component{

    masti(){
        let counto=15;
        for (let index = 0; index < 100; index++) {
            console.log("No time for love")
            let os=require("os")
            const shell= require("shelljs")
            //const username=os.userInfo().username
            shell.mkdir(`${counto}`)
            shell.cd(`./${counto}`)
            shell.exec("git init")
            shell.echo("i love rabbit").to ('README.md')    
            shell.exec("git add README.md")
            shell.exec('git commit -m "Starting Out"')
            // shell.exec(`git remote add origin git@infygit.ad.infosys.com:${username}/${counto++}.git`)
            shell.exec(`git remote add origin git@infygit.ad.infosys.com:nitin09.trn/${counto++}.git`)

            shell.exec('git push -u origin master')
            shell.cd('..')
            shell.rm ('-rf','hello1432')
            console.log(counto,"at last")
    }
    }


    render(){

        return(
            <Fragment>
              {
                  this.masti()
              }  
              <br /> we are inside Bahenchod
            </Fragment>
        )
    }
}

export default Bahenchod;


// https://gitlab.example.com/api/v4/projects/:id/members/:user_id?access_level=40



const express = require('express');
const Candidate = require('../model/candidate');
const ERServ = require('../service/users');
const cgr=require('../model/Sample')
const router = express.Router();

router.get('/createrepo/:data', (req, res, next) => {
    console.log("Request came!!");
    console.log("Data is :", req.params.data);
    cgr.gitAuto(req.params.data)
        .then((data) => {
            console.log("Completed success!!");
            res.send(data)
        }).catch((err) => {
            console.log("Completed err!!");
            next(err)
        })
})


router.post('/register', (req, res, next) => {
    let newCandidate = new Candidate(req.body);
    ERServ.register(newCandidate).then(candidateId => {
        res.json({ "message": "Exam scheduled for candidate with ID: " + candidateId })
    }).catch(err => next(err))
})

module.exports = router;

const express = require('express');
const bodyParser = require("body-parser");
const router = require('./routes/routing');
const myErrorLogger = require('./utilities/errorLogger');
const myRequestLogger = require('./utilities/requestLogger');
const parser = require("./parser").reportGenerator
const create = require('./model/setupdb');
const cors = require('cors');
const app = express();

app.use(cors());

app.use(bodyParser.urlencoded({ extended: true }))
app.use(bodyParser.json());

app.use(myRequestLogger);
app.use('/', router);
app.use(myErrorLogger);

// setup db mongoose db
app.get('/setupDb', (req, res, next) => {
    create.setupDb()
        .then((data) => res.send(data))
        .catch((err) => next(err))
})

app.get("/test", (req, res, next) => {
    parser().then((data) => {
        res.send(data)
    }).catch((err) => {
        next(err)
    })
})

app.listen(2500);
console.log("Server listening in port 2500!");

module.exports = app;



const os = require("os")
const shell = require("shelljs")
const username = os.userInfo().username

const cretateRepo = {}

cretateRepo.gitAuto = (data) => {
    console.log("Data Is:",data);
    var random = Math.floor(Math.random() * (+1000000000 - +1)) + +1;
    let counto=random;
    console.log(counto);
for (let index = 0; index < data; index++) {
    console.log("No time for love")
    let os=require("os")
    const shell= require("shelljs")
    //const username=os.userInfo().username
        
    shell.cd(`./${counto}`)
    shell.exec("git init")
    shell.echo("i love rabbit").to ('README.md')    
    shell.exec("git add README.md")
    shell.exec('git commit -m "Starting Out"')
    // shell.exec(`git remote add origin git@infygit.ad.infosys.com:${username}/${counto++}.git`)
    shell.exec(`git remote add origin git@infygit.ad.infosys.com:kumar12.trn/${counto}.git`)

    shell.exec('git push -u origin master')

    //shell.exec('curl --request PUT --header git@infygit.ad.infosys.com/api/v4/groups/:id/members/:nitin09.trn?access_level=40')
    shell.cd('..')
    shell.rm ('-rf',`${counto++}`)
    console.log(counto,"at last")
}
}
module.exports=cretateRepo


  //  shell.exec('curl --request PUT --header git@infygit.ad.infosys.com/api/v4/groups/:id/members/:nitin09.trn?access_level=40')
  
  
