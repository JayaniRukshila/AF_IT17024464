
---------/// server js ////--------
const Bundler = require('parcel-bundler'),
    express = require('express'),
    mongoose = require('mongoose');

const bundler = new Bundler('./public/index.html', {});

const app = express();

app.use(express.json());

mongoose.connect('mongodb://localhost:27017/react-sample').then(() => {
    console.log('Connected to the DB');
}).catch(err => {
    console.error(err);
    process.exit(-1);
});

app.use('/api/messages', require('./app/routes/message.server.routes'));
app.use('/api/course',require('./app/routes/course/course.server.routes'));
app.use('/api/subject', require('./app/routes/subject/subject.server.routes'));

app.use(bundler.middleware());

app.use(express.static('./dist'));

app.get('/', function (req, res) {
    res.sendFile('./dist/index.html');
});

app.listen(5000, err => {
    if (err) {
        console.error(err);
        return;
    }

    console.log('Application is running on port 3000');
});

------//// public/index jsx ////----------
import React from 'react';
import {render} from 'react-dom';

import 'bootstrap/dist/css/bootstrap.min.css';
import { BrowserRouter } from 'react-router-dom';

import App from './App';

render(<BrowserRouter><App/></BrowserRouter>, document.getElementById('app'));

---------------//// public/index html ////-----
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>React App</title>
</head>
<body>
<div class="container" id="app"></div>
<script src="./index.jsx"></script>
</body>
</html>

-------------//// public/Appjsx ////--------
import React, { Component } from 'react';
import { BrowserRouter as Router, Route, Link, Switch } from 'react-router-dom';
import Insert from './components/course/insert/insert';
import View from './components/course/view/view';

export default class App extends Component {
    constructor(props) {
        super(props);
        this.state = {
            message: ''
        }
    }

    componentDidMount() {
        fetch('/api/messages', { method: 'GET' })
            .then(res => res.json())
            .then(jsonRes => {
                this.setState({ message: jsonRes.message });
            })
            .catch(err => {
                this.setState({ message: 'An error occurred' });
            });
    }

    render() {
        return (
            <Router>
                <div className="container">
                    <h2>{this.state.message}</h2>
                    <Link style={{marginRight:"10px"}} to={"/insert"}>Insert</Link>
                    <Link to={"/View"}>View</Link>
                    <Switch>
                        <Route path="/insert" component={Insert} ></Route>
                        <Route path="/View" component={View} ></Route>
                    </Switch>
                </div>
            </Router>
        )
    }
}

---------/// public/components/course/insert/insert jsx ////-----
import React, { Component } from 'react';
import axios from 'axios';

class Insert extends Component {
    constructor() {
        super();
        this.state = {
            name: '',
            code: '',
            passMark: '',
            lectureInCharge: '',
            subjects: [],
            subject: ''
        }
        this.handleInputChange = this.handleInputChange.bind(this);
        this.handleSubmit = this.handleSubmit.bind(this);
    }

    handleInputChange(e) {
        this.setState({
            [e.target.name]: e.target.value
        })
    }

    componentDidMount() {
        axios.get('/api/subject').then(
            data => {
                this.setState({
                    subjects: data.data
                })
            }
        )
    }

    handleSubmit(e) {
        e.preventDefault();
        const obj = {
            name: this.state.name,
            code: this.state.code,
            passMark: this.state.passMark,
            lectureInCharge: this.state.lectureInCharge,
            subject: this.state.subject
        }
        console.log(JSON.stringify(obj));

        axios.post('/api/course/insert', obj).then(
            data => {
                console.log('Success ' + data.data);
            }
        )

        this.setState({
            name: '',
            code: '',
            passMark: '',
            lectureInCharge: '',
            subject: ''
        })
    }
    render() {
        return (
            <div className="container">
                <form onSubmit={this.handleSubmit}>
                    <div className="form-group">
                        <label>Name</label>
                        <input
                            type="text"
                            className="form-control"
                            onChange={this.handleInputChange}
                            value={this.state.name}
                            name="name"
                        />
                    </div>
                    <div className="form-group">
                        <label>Code</label>
                        <input
                            type="text"
                            className="form-control"
                            onChange={this.handleInputChange}
                            value={this.state.code}
                            name="code"
                        />
                    </div>
                    <div className="form-group">
                        <label>Pass Mark</label>
                        <input
                            type="text"
                            className="form-control"
                            onChange={this.handleInputChange}
                            value={this.state.passMark}
                            name="passMark"
                        />
                    </div>
                    <div className="form-group">
                        <label>Lecture In Charge</label>
                        <input
                            type="text"
                            className="form-control"
                            onChange={this.handleInputChange}
                            value={this.state.lectureInCharge}
                            name="lectureInCharge"
                        />
                    </div>
                    <div className="form-group">
                        <label>Subject</label>
                        <select name="subject" className="form-control" onChange={this.handleInputChange} value={this.state.subject}>
                            {
                                this.state.subjects.map(sub => {
                                    return (
                                        <option key={sub._id} value={sub._id}>{sub.name}</option>
                                    )
                                })
                            }
                        </select>
                    </div>
                    <div className="form-group">
                        <input type="submit" value="submit" />
                    </div>
                </form>
            </div>
        )
    }
}

export default Insert

---------------------//// public/components/course/view/view jsx ////-----------
import React, { Component } from 'react';
import Axios from 'axios';

class View extends Component {
    constructor(){
        super();
        this.state = {
            courses: [],
            subjects: []
        }
    }

    componentDidMount(){
        Axios.get('/api/course').then(
            data => {
                this.setState({
                    courses: data.data
                })
            }
        )
    }

    subject(id){
        Axios.get('/api/subject/find/' + id).then(
            dataSet => {
                alert('Name : ' + dataSet.data.name + ' Description : ' + dataSet.data.description);
            }
        )
    }
    render() {
        return (
            <div className="container">
                <table className="table">
                    <thead>
                        <th>Name</th>
                        <th>Code</th>
                        <th>Pass Mark</th>
                        <th>Lecture In Charge</th>
                        <th>Subjects</th>
                    </thead>
                    <tbody>
                        {
                            this.state.courses.map( cou => {
                                return (
                                    <tr key={cou._id}>
                                        <td>{cou.name}</td>
                                        <td>{cou.code}</td>
                                        <td>{cou.passMark}</td>
                                        <td>{cou.lectureInCharge}</td>
                                        <td><button onClick = {this.subject.bind(this,cou.subject)}>Subjects</button></td>
                                    </tr>
                                )
                            })
                        }
                    </tbody>
                </table>
            </div>
        )
    }
}

export default View

---------------/// app/models/Course js /// ------------
const mongoose = require('mongoose');
const Schema = mongoose.Schema;
const subject = require('./Subject');

const courseSchema = new Schema({
    name: {
        type: String,
        required: true
    },
    code: {
        type: String,
        required: true
    },
    passMark : {
        type: String,
        required: true
    },
    lectureInCharge: {
        type: String,
        required: true
    },
    subject: [
        {
            type: Schema.Types.ObjectId,
            ref: 'subject'
        }
    ]
});

module.exports = mongoose.model('course', courseSchema);

----------------/// app/models/Subject js ///--------
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const subjectSchema = new Schema({
    name: {
        type: String,
        required: true
    },
    description: {
        type: String,
        required: true
    },
    amount: {
        type: String,
        required: true
    }
});

module.exports = mongoose.model('subject',subjectSchema);

----------------/// app/routes/course/ course.server.routes js ///-----------
const express = require('express');
const router = express.Router();
const course = require('../../models/Course');

router.post('/insert', function(req,res){
    const courseObj = new course(req.body);
    courseObj.save().then(
        data => {
            return res.status(200).json(data);
        }
    ).catch(
        err => {
            return res.status(400).json(err);
        }
    )
});

router.get('/', function(req,res){
    course.find().then(
        data => {
            return res.status(200).json(data);
        }
    ).catch(
        err => {
            return res.status(400).json(err);
        }
    )
});

router.get('/find/:id', function(req,res){
    course.findById(req.params.id).then(
        data => {
            return res.status(200).json(data);
        }
    ).catch(
        err => {
            return res.status(400).json(err);
        }
    )
});

module.exports = router;

--------------------/// app/routes/subject/ message.server.routes js///----------------
const express = require('express');

const Router = express.Router();

Router.get('/', function (req, res) {
    res.json({message: 'Welcome to the react + express + mongoDB'});
});

module.exports = Router;

--------------// SPRING //--------------
mvn clean install
java -jar target/affinal-0.0.1-SNAPSHOT.jar
 <dependency> 
    	<groupId>io.springfox</groupId> 
    	<artifactId>springfox-swagger2</artifactId> 
    	<version>2.9.2</version> 
    </dependency> 
    <dependency> 
    	<groupId>io.springfox</groupId> 
    	<artifactId>springfox-swagger-ui</artifactId>
    	<version>2.9.2</version>
     </dependency>

  // com.af.demo
package com.example.demo;


import org.springframework.boot.SpringApplication;

import org.springframework.boot.autoconfigure.SpringBootApplication;


import springfox.documentation.swagger2.annotations.EnableSwagger2;



@SpringBootApplication

@EnableSwagger2

public class DemoApplication {

	
	public static void main(String[] args) {

		SpringApplication.run(DemoApplication.class, args);
		
		
	
	}	


}


  // pkg controller/CourseController.java --------
import java.util.ArrayList;

import java.util.List;


import org.springframework.beans.factory.annotation.Autowired;

import org.springframework.http.HttpStatus;

import org.springframework.http.ResponseEntity;

import org.springframework.web.bind.annotation.CrossOrigin;

import org.springframework.web.bind.annotation.PathVariable;

import org.springframework.web.bind.annotation.RequestMapping;

import org.springframework.web.bind.annotation.RequestMethod;

import org.springframework.web.bind.annotation.RestController;


import com.sliit.af2018.models.Course;

import com.sliit.af2018.models.Subject;

import com.sliit.af2018.service.CourseService;

import com.sliit.af2018.service.SubjectService;



@RestController

@RequestMapping("/course")

@CrossOrigin(origins="*")

public class CourseController {

	
	@Autowired
	
	private CourseService courseService;
	
	@Autowired
	
	private SubjectService subjectService;
	
	

	@RequestMapping(method= RequestMethod.GET , value="/calculate/{id}")
	
	public ResponseEntity<Integer> calculate(@PathVariable("id") String id){
		
		Course course = courseService.find(id);
		
		int amount = 0;
		
		List<String> subject_id = new ArrayList<>();
		
		subject_id = course.getSubject();
		
		for(int i = 0 ; i < subject_id.size() ; i++) {
			
			String subId = subject_id.get(i);
			
			Subject subject = subjectService.find(subId);
			
			amount += Integer.parseInt(subject.getAmount());
		
		}
		
			return new ResponseEntity<Integer>(amount , HttpStatus.OK);
	
		}
	

}

    //pkg models/Course.java ------
package com.sliit.af2018.models;


import java.util.List;


import org.springframework.data.mongodb.core.mapping.Document;


import com.fasterxml.jackson.annotation.JsonIgnoreProperties;



@Document(collection = "courses")

public class Course {
	

	private String name;
	
	private String code;
	
	private String passMark;
	
	private String lectureInCharge;
	

	private List<String> subject;

	

	public String getName() {

		return name;
	
	}

	

	public void setName(String name) {
	
		this.name = name;
	
	}

	

	public String getCode() {
		
		return code;
	
	}

	

	public void setCode(String code) {
	
		this.code = code;
	
	}

	
	
	public String getPassMark() {
	
		return passMark;
	
	}

	

	public void setPassMark(String passMark) {
	
		this.passMark = passMark;
	
	}

	
	
	public String getLectureInCharge() {

		return lectureInCharge;
	
	}

	

	public void setLectureInCharge(String lectureInCharge) {
	
		this.lectureInCharge = lectureInCharge;
	
	}

	
	
	public List<String> getSubject() {
	
		return subject;
	
	}

	

	public void setSubject(List<String> subject) {
	
		this.subject = subject;
	
	}


}


      // pkg models/Subject.java
package com.sliit.af2018.models;


import org.springframework.data.mongodb.core.index.Indexed;

import org.springframework.data.mongodb.core.mapping.Document;



@Document(collection = "subjects")

public class Subject {
	
	@Indexed
	
	private String _id;
	
	private String name;
	
	private String description;
	
	private String amount;
	

	
	public String get_id() {
		
		return _id;
	
	}

	

	public void set_id(String _id) {

		this._id = _id;
	
	}

	

	public String getName() {

		return name;
	
	}

	

	public void setName(String name) {

		this.name = name;
	
	}

	

	public String getDescription() {

		return description;
	
	}

	

	public void setDescription(String description) {
	
		this.description = description;

	}

	

	public String getAmount() {
		
		return amount;
	
	}

	

	public void setAmount(String amount) {
	
		this.amount = amount;
	
	}


}


      // pkg repository/CourseRepository.java interface
package com.sliit.af2018.repository;


import org.springframework.data.mongodb.repository.MongoRepository;

import org.springframework.stereotype.Repository;


import com.sliit.af2018.models.Course;



@Repository
public interface CourseRepository extends MongoRepository<Course, String>{


}


     // pkg repository/SubjectRepository.java interface
package com.sliit.af2018.repository;


import org.springframework.data.mongodb.repository.MongoRepository;

import org.springframework.stereotype.Repository;


import com.sliit.af2018.models.Subject;



@Repository
public interface SubjectRepository extends MongoRepository<Subject, String>{
	

}


    // pkg service/AF2018Application.java
package com.sliit.af2018;


import org.springframework.boot.SpringApplication;

import org.springframework.boot.autoconfigure.SpringBootApplication;



@SpringBootApplication
public class Af2018Application {

	
	public static void main(String[] args) {

		SpringApplication.run(Af2018Application.class, args);
	
	}


}


