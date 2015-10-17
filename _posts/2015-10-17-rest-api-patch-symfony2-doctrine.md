---
layout:     post
title:      Patching Entities with Doctrine and Symfony2 Forms
date:       2015-10-17
summary:    Developing RESTful Symfony2 applications with Services and Forms.
categories: blog rest doctrine orm php
---

We are building out a Laboratory Information Management System (LIMS) for a clinical genetics company. Most of our applications are data grid heavy with several partial updates using RESTful calls over AJAX. Our stack is CentOS, Apache 2.2, PHP 5.6, Symfony 2.7, Doctrine 2, PostgreSQL 9.4 and the front end is a modular JavaScript application using NPM, webpack and BackboneJS. We are a small team (just 2 full stack developers and 1 backend bioinformatics scientist). For this reason we kept our stack light and easy with commonly used well documented tools.

Since our data is mostly imported via automated imports through backend services from our laboratory we seldom actually CREATE data from the application's front end. We never fully replace resources either so PUT is a seldom used method. With that in mind I set out to architect a simple yet elegant way to update our entities using the HTTP PATCH method with JMSSerializerBundle and the FOSRestBundle. Here I would like to share a service I created that simple accepts a hydrated FormType method. It will validate the Form using the Entity's validation rules and store the data if it passes validation.

Here is a FOSRestBundle API endpoint which receives a simple HTTP PATCH payload:

{% highlight json %}
{ 
  "appbundle_samplecnv": { 
    "comments": "test",
    "cnv": {
      "chrBgn": "41256136", 
      "chrEnd": "41256137", 
      "cVariant": "exon4-6 del", 
      "comments": "test"
    }
  }
}
{% endhighlight %}

The above JSON payload will be sent to our Symfony2 FOSRestBundle endpoint using the PATCH method. Notice the first key of the object is the actual name of our Symfony2 form. This is important. We'll leverage Symfony2's ParamConverter, create a Form instance, and importantly, set the method to PATCH. This allows use to use the handleReqeust method inside our service, isntead of the depracated <em>submit</em> method while preserving validation.

{% highlight php %}
public function patchSampleCnv(SampleCnv $sampleCnv)
{
    $mgr = $this->get('app.sample_cnv_manager');
    $api = $this->get('app.api_manager');
    $patchForm = $this->createForm(new SampleCnvType, $sampleCnv, [
        'method' => 'PATCH'
    ]);
    $form = $mgr->patch($patchForm);
    if ($form === false) {
        throw new HttpException(400, "SampleCnv could not be saved.");
    }
    return $api->createFormResponse($form);
}
{% endhighlight %}

Our EntityManager service will handle the form validation keeping our controller thin and removing business logic. The patch service will be resused numerous times in our application. We simply need to maintain the Entity, its validation rules and wire it to our RESTful controllers.

{% highlight php %}
public function patch(Form $form)
{
    $request = $this->getRequest();
    $form->handleRequest($request);
    $formData = $form->getData();

    $validator = $this->container->get('validator');
    $errors = $validator->validate($form);
    foreach ($errors as $error) {
        $form->addError(new \Symfony\Component\Form\FormError($error->getMessage()));
    }
    if (!$form->isValid()) {
        return $form;
    }
    try {
        $this->em->persist($formData);
        $this->em->flush();
    } catch (ORMException $e) {
        $this->getLogger()->error($e->getMessage());
        return false;
    }
    return $form;
}
{% endhighlight %}

The above snippet will honor all Entity level validation rules such as this:
{% highlight php %}
/**
 * @var integer
 *
 * @ORM\Column(name="chr_bgn", type="integer")
 * @Gedmo\Versioned
 *
 * @Assert\Expression(
 *     "value < this.getChrEnd()",
 *     message="Chr begin must be less than chr end."
 * )
 */
private $chrBgn;
{% endhighlight %}

Our response back from the server when errors are present will simply resemble the following:
{% highlight json %}
{
    "success": false,
    "errors": [
        "Chr begin must be less than chr end.",
        "Chr end must be greater than chr start."
    ],
    "type": "appbundle_samplecnv",
    "data": {
      "id": 8,
      "...": "..."
    }
}
{% endhighlight %}

You may have noticed I also utilize a custom response manager for my api output. I will share that bit below. It takes the Form object you got back from the manager, checks for errors and automatically formats the JSON response. We also heavily use the JMSSerializer bundle and create several VirtualProperties for user friendly output on certain Entity fields. I highly recommend it.

{% highlight php %}
/**
 * @param Symfony\Component\Form\Form $form
 * @return FOS\RestBundle\View\View
 */
public function createFormResponse(\Symfony\Component\Form\Form $form)
{
    $errors = [];
    foreach($form->getErrors() as $error) {
        $errors[] = $error->getMessage();
    }
    $this->view->setData(new ArrayCollection([
        'success' => $form->isValid(),
        'errors'  => $errors,
        'type'    => $form->getName(),
        'data'    => $form->getData(),
    ]));
    return $this->view;
}
{% endhighlight %}

That's really about it. As you can see it would be trivial from here to provide a CREATE, PUT and DELETE into the EntityManagerService. I'll leave that for you as a homework assignment. Of course if you have any questions don't hesitate to leave a comment or tweet me @tmpjrdotme. I will leave a more complete gist below.

{% gist 78f3b66e547cce640200 %}
